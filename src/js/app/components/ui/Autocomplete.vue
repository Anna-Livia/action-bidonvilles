<template>
    <ValidationProvider
        ref="provider"
        :rules="rules"
        :name="validationName || label"
        v-slot="{ errors }"
        :vid="id"
    >
        <InputWrapper :hasErrors="!!errors.length">
            <InputLabel :label="label" :info="info" />
            <AutocompleteVue
                :search="search"
                :default-value="searchInput"
                :placeholder="placeholder"
                :aria-label="placeholder"
                :getResultValue="getResultValue"
                :debounceTime="debounceTime"
                @submit="onItemSelect"
            >
                <template
                    #default="{
                rootProps,
                inputProps,
                inputListeners,
                resultListProps,
                resultListListeners,
                results,
                resultProps
              }"
                >
                    <div v-bind="rootProps">
                        <div class="relative">
                            <InputIcon
                                class="text-primary pl-6"
                                position="before"
                                :icon="prefixIcon"
                                v-if="prefixIcon"
                            />

                            <input
                                ref="searchInput"
                                v-bind="inputProps"
                                v-on="inputListeners"
                                :class="classes"
                                @focus="handleFocus"
                                @blur="handleBlur"
                                v-model="searchInput"
                                class="pl-12 pr-12"
                            />
                            <InputIcon position="after" class="pr-6">
                                <Spinner v-if="loading" />
                                <div
                                    @click="removeItem"
                                    class="cursor-pointer "
                                >
                                    <Icon
                                        class="text-primary text-display-sm"
                                        v-if="
                                            !loading &&
                                                value &&
                                                getResultValue(value) ===
                                                    searchInput
                                        "
                                        icon="times"
                                        data-cy-button="clear"
                                    />
                                </div>
                            </InputIcon>
                        </div>

                        <transition name="fade">
                            <div
                                v-if="focused && searchInput"
                                :class="[
                                    'origin-top-left-10 absolute z-10 left-0 mt-2 w-full rounded-md shadow-lg'
                                ]"
                            >
                                <slot
                                    :results="results"
                                    :resultListProps="resultListProps"
                                    :resultListListeners="resultListListeners"
                                    :resultProps="resultProps"
                                    :getResultValue="getResultValue"
                                >
                                    <Menu v-if="!results.length">
                                        <MenuItem>
                                            Aucun résultat
                                        </MenuItem>
                                    </Menu>
                                    <Menu
                                        v-else
                                        v-bind="resultListProps"
                                        v-on="resultListListeners"
                                    >
                                        <MenuItem
                                            v-for="(result, index) in results"
                                            :key="resultProps[index].id"
                                            v-bind="resultProps[index]"
                                            :class="[
                                                'cursor-pointer',
                                                resultProps[index][
                                                    'aria-selected'
                                                ] && 'bg-gray-100'
                                            ]"
                                        >
                                            <div>
                                                {{ getResultValue(result) }}
                                            </div>
                                        </MenuItem>
                                    </Menu>
                                </slot>
                            </div>
                        </transition>
                        <div>
                            <slot name="extra" :removeItem="removeItem" />
                        </div>
                    </div>
                </template>
            </AutocompleteVue>
            <InputError>{{ errors[0] }}</InputError>
        </InputWrapper>
    </ValidationProvider>
</template>

<script>
import InputLabel from "./Form/utils/InputLabel.vue";
import InputWrapper from "./Form/utils/InputWrapper.vue";
import InputError from "./Form/utils/InputError.vue";
import getInputClasses from "./Form/utils/getInputClasses";
import InputIcon from "./Form/utils/InputIcon.vue";

export default {
    components: {
        InputLabel,
        InputWrapper,
        InputError,
        InputIcon
    },
    props: {
        loading: {
            type: Boolean
        },
        label: {
            type: String
        },
        info: {
            type: String
        },
        validationName: {
            type: String
        },
        rules: {
            type: String
        },
        id: {
            type: String
        },
        inputClasses: {
            type: Array,
            default: () => []
        },
        search: {
            type: Function,
            required: true
        },
        defaultValue: {
            type: Object,
            required: false
        },
        placeholder: {
            type: String
        },
        getResultValue: {
            type: Function,
            default: val => val
        },
        debounceTime: {
            type: Number,
            default: 0
        },
        variant: {
            type: String,
            default: "default"
        },
        prefixIcon: {
            type: String
        }
    },
    computed: {
        classes() {
            const inputOptions = {
                error: this.error,
                prefixIcon: this.prefixIcon
            };

            const defaultClasses = {
                state: [...getInputClasses("state", inputOptions)],
                default: getInputClasses("default", inputOptions)
            }[this.variant];
            return [...this.inputClasses, ...defaultClasses];
        }
    },
    data() {
        return {
            show: true,
            focused: false,
            value: this.defaultValue || null,
            searchInput: this.defaultValue
                ? this.getResultValue(this.defaultValue)
                : "",
            results: []
        };
    },
    methods: {
        removeItem() {
            this.value = null;
            this.searchInput = "";
            this.$emit("submit", null);
            this.$emit("input", null);
            this.$refs.provider.syncValue(null);
            this.$refs.provider.validate();
        },
        onItemSelect(newValue) {
            // Update local new value & Emit
            this.value = newValue;
            // If user has selected an item, update search input
            if (newValue) {
                this.searchInput = this.getResultValue(newValue);
            }

            this.$emit("submit", newValue);
            this.$emit("input", newValue);
            this.$refs.provider.syncValue(newValue);
            this.$refs.provider.validate();
            this.$refs.searchInput.blur();
        },
        handleFocus() {
            this.focused = true;
        },

        handleBlur() {
            // If user has deleted his input, delete the selected value
            if (!this.value) {
                this.onItemSelect(null);
            } else {
                // If user has changed his last input, restore to last value
                this.searchInput = this.getResultValue(this.value);
            }
            this.focused = false;
        }
    }
};
</script>
