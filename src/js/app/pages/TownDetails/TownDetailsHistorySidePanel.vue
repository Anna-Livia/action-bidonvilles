<template>
    <SidePanel
        :isOpen="isOpen"
        :closePanel="closePanel"
        :closeClickOutside="true"
    >
        <div class="  px-8 ">
            <div class="border-b-2 border-G200 py-4">
                <div class="flex justify-end">
                    <Button
                        variant="primaryText"
                        icon="times"
                        size="lg"
                        @click="closePanel"
                    />
                </div>

                <div class="flex items-center text-primary">
                    <Icon icon="history" class="mr-2 " />
                    <div class="text-display-md text-primary">
                        Historique des modifications
                    </div>
                </div>
            </div>

            <div>
                <div class="text-sm font-bold my-4">
                    {{ town.changelog.length }} modification{{
                        town.changelog.length > 1 ? "s" : ""
                    }}
                </div>

                <div
                    v-for="changelog in town.changelog"
                    :key="changelog.id"
                    class="py-4 border-b-2 border-G200"
                >
                    <div class="text-sm text-G600">
                        {{ formatDate(changelog.date, "d M y à h:i") }}
                    </div>

                    <div class="text-primary">
                        <router-link
                            :to="
                                `/annuaire/${changelog.author.organization.id}`
                            "
                        >
                            <div class="flex items-center">
                                <font-awesome-icon
                                    icon="user"
                                    class="w-8 text-md"
                                ></font-awesome-icon>
                                <div class="font-bold ml-2">
                                    {{ changelog.author.first_name }}
                                    {{ changelog.author.last_name }}
                                </div>
                            </div>
                        </router-link>
                    </div>
                    <div>
                        <div
                            v-for="diff in changelog.diff"
                            :key="diff.field"
                            class="changelogContent my-2"
                        >
                            <div class="text-green">{{ diff.field }}</div>
                            <div>
                                <span class="line-through">{{
                                    diff.oldValue || "non renseigné"
                                }}</span>
                                <span
                                    >,
                                    {{ diff.newValue || "non renseigné" }}
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </SidePanel>
</template>

<script>
export default {
    props: {
        town: {
            type: Object
        },
        isOpen: {
            type: Boolean
        },
        closePanel: {
            type: Function
        }
    },

    methods: {
        /**
         * @see index.js
         */
        formatDate(...args) {
            return window.App.formatDate.apply(window, args);
        }
    }
};
</script>

<style>
.changelogContent {
    margin-left: 22px;
}
</style>
