<template>

    <div v-if="!isEmpty || data.linkTo" :class="[ data.type, { 'mb-8': isTopLevel }, 'select-none' ]">

        <component v-if="data.label && isTopLevel" v-bind="topLevelLink"
                   @click="toggleTopLevel"
                   :class="{ 'cursor-pointer': isTopCollapsible, 'select_light': isShow }"
                   style="cursor: pointer;"
                   class="flex flex-1 items-center font-normal text-white mb-2 text-base no-underline relative dim">

            <div v-if="data.icon" class="sidebar-icon" v-html="data.icon"/>

            <svg v-else class="sidebar-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
                <path fill="var(--sidebar-icon)"
                      d="M3 1h4c1.1045695 0 2 .8954305 2 2v4c0 1.1045695-.8954305 2-2 2H3c-1.1045695 0-2-.8954305-2-2V3c0-1.1045695.8954305-2 2-2zm0 2v4h4V3H3zm10-2h4c1.1045695 0 2 .8954305 2 2v4c0 1.1045695-.8954305 2-2 2h-4c-1.1045695 0-2-.8954305-2-2V3c0-1.1045695.8954305-2 2-2zm0 2v4h4V3h-4zM3 11h4c1.1045695 0 2 .8954305 2 2v4c0 1.1045695-.8954305 2-2 2H3c-1.1045695 0-2-.8954305-2-2v-4c0-1.1045695.8954305-2 2-2zm0 2v4h4v-4H3zm10-2h4c1.1045695 0 2 .8954305 2 2v4c0 1.1045695-.8954305 2-2 2h-4c-1.1045695 0-2-.8954305-2-2v-4c0-1.1045695.8954305-2 2-2zm0 2v4h4v-4h-4z"/>
            </svg>

            <Badge :label="data.badge" :dim="isTopCollapsible || data.linkTo">

                <span class="flex text-white sidebar-label">
                    {{ data.label }}
                </span>

                <CollapsibleIndicator :expanded="topExpanded" :visible="isTopCollapsible"/>

            </Badge>


            <svg v-if="!isShow" style="margin-right: 0; width: 1rem; height:1rem;" t="1602317232981" class="sidebar-icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="8364">
                <path d="M275.2 6.4c6.4 0 19.2 0 25.6 6.4l473.6 473.6c6.4 6.4 6.4 12.8 6.4 25.6 0 6.4-6.4 19.2-6.4 25.6l-473.6 473.6c-12.8 12.8-32 12.8-44.8 0-12.8-12.8-12.8-32 0-44.8L704 512 256 64C243.2 51.2 243.2 32 256 19.2c0-12.8 12.8-12.8 19.2-12.8z" fill="var(--sidebar-icon)" p-id="8365">
                </path>
            </svg>

            <svg v-else style="margin-right: 0; width: 1rem; height:1rem;" t="1602317859146" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="8509">
                <path d="M512 680.521143l-374.491429-411.794286a36.571429 36.571429 0 0 0-55.588571 47.616l404.918857 447.268572a36.571429 36.571429 0 0 0 57.563429-2.56l399.652571-447.195429a36.571429 36.571429 0 1 0-59.538286-42.569143L512 680.521143z" fill="var(--sidebar-icon)" p-id="8510">
                </path>
            </svg>

        </component>

        <CollapseTransition :duration="150" v-if="isShow">

            <ResourceList class="resources-only"
                          v-if="isTopLevel && data.resources.length && (!isTopCollapsible || topExpanded)"
                          :resources="data.resources"
                          :recursive="recursive"
                          :remember-menu-state="rememberMenuState"/>

        </CollapseTransition>

        <template v-if="isGroup && data.resources.length">

            <h4 class="relative select-none ml-8 mt-4 text-xs text-white-50% uppercase tracking-wide cursor-pointer"
                v-if="data.label"
                @click="toggleGroup(data.id)">

                <CollapsibleIndicator :expanded="activeMenu[data.id]" :visible="isTopCollapsible"/>

                <Badge :label="data.badge">
                    {{ data.label }}
                </Badge>

            </h4>

            <CollapseTransition :duration="150">

                <ResourceList v-if="activeMenu[data.id]"
                              :resources="data.resources"
                              :recursive="recursive"
                              :remember-menu-state="rememberMenuState"/>

            </CollapseTransition>

        </template>

    </div>

</template>
<style>
    .select_light{
        opacity: 0.6;
    }
</style>

<script>

    import { CollapseTransition } from 'vue2-transitions'
    import ResourceList from './ResourceList'
    import Badge from './Badge'
    import CollapsibleIndicator from './CollapsibleIndicator'

    export default {
        name: 'CollapsibleResourceManager',
        components: { CollapsibleIndicator, CollapseTransition, ResourceList, Badge },
        props: {
            data: { type: Object, required: true },
            rememberMenuState: { type: Boolean, default: false },
            recursive: { type: Boolean, default: false }
        },
        data() {
            return {
                topExpanded: this.data.expanded,
                activeMenu: { [ this.data.id ]: this.data.expanded },
                isShow: true
            }
        },
        created() {

            if (this.rememberMenuState) {

                const state = localStorage.getItem(this.cacheKey)

                if (state) {

                    this.activeMenu[ this.data.id ] = JSON.parse(state)

                }

                this.$watch(() => this.activeMenu[ this.data.id ], state => {

                    localStorage.setItem(this.cacheKey, state)

                })

            }

        },
        computed: {
            isTopCollapsible() {
                return this.data.expanded === null ? false : true && this.data.linkTo === null
            },
            isGroup() {
                return this.data.type === 'group'
            },
            isTopLevel() {
                return this.data.type === 'top_level'
            },
            cacheKey() {
                return `menu-state.${ this.data.id }`
            },
            isEmpty() {
                return this.data.resources.length === 0
            },
            topLevelLink() {

                if (this.data.linkTo) {

                    return {
                        is: 'router-link',
                        to: this.data.linkTo.router,
                        target: this.data.linkTo.target,
                        class: [ 'cursor-pointer', 'dim' ]
                    }

                }

                return {
                    is: 'h3'
                }

            }
        },
        methods: {
            toggleTopLevel() {
                // console.log(this.isShow);
                this.isShow = !this.isShow
                if (this.isTopCollapsible) {
                    this.topExpanded = !this.topExpanded
                }
            },
            toggleGroup(id) {
                this.activeMenu[ id ] = !this.activeMenu[ id ]
            }
        }
    }

</script>

<style scoped>

    .top_level ul li:first-child {
        padding-top: 0;
    }

    .group ul li:first-child {
        padding-top: 1rem;
    }

    .group h4 {
        margin-left: 0;
    }

    .group h4:first-child {
        margin-top: 0;
    }

</style>
