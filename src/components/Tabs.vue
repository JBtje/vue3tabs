<template>
    <div :class="wrapperClass"
         id="tabs-panels"
    >
        <ul role="tablist"
            :class="navClass"
        >
            <li v-for="(tab, i) in tabs"
                :key="i"
                :class="[ navItemClass, tab.isDisabled ? navItemDisabledClass : '', tab.isActive ? navItemActiveClass : '' ]"
                role="presentation"
                @click="selectTab(tab.hash, $event)"
            >
                <a v-html="tab.header"
                   :aria-controls="tab.hash"
                   :aria-selected="tab.isActive"
                   :href="tab.hash"
                   :class="[ navItemLinkClass, tab.isDisabled ? navItemLinkDisabledClass : '', tab.isActive ? navItemLinkActiveClass : '' ]"
                   role="tab"
                ></a>
            </li>
        </ul>
        <div :class="panelsWrapperClass"
             role="tabpanel"
             ref="scrollWindow"
        >
            <slot/>
        </div>
    </div>
</template>

<script>
import expiringStorage                             from '../expiringStorage';
import {onMounted, provide, reactive, ref, toRefs} from 'vue';

export default {
    props: {
        cacheLifetime:            {
            type:    Number,
            default: 5,
        },
        navClass:                 {
            type:    String,
            default: 'tabs-component-tabs',
        },
        navItemActiveClass:       {
            type:    String,
            default: 'is-active',
        },
        navItemClass:             {
            type:    String,
            default: 'tabs-component-tab',
        },
        navItemDisabledClass:     {
            type:    String,
            default: 'is-disabled',
        },
        navItemLinkActiveClass:   {
            type:    String,
            default: 'is-active',
        },
        navItemLinkClass:         {
            type:    String,
            default: 'tabs-component-tab-a',
        },
        navItemLinkDisabledClass: {
            type:    String,
            default: 'is-disabled',
        },
        options:                  {
            type:     Object,
            required: false,
            default:  () => ({
                useUrlFragment: true,
                defaultTabHash: null,
            }),
        },
        panelsWrapperClass:       {
            type:    String,
            default: 'tabs-component-panels',
        },
        wrapperClass:             {
            type:    String,
            default: 'tabs-component',
        },
    },

    emits: ['changed', 'clicked'],

    setup( props, context ) {
        const state = reactive( {
            activeTabHash:     '',
            lastActiveTabHash: '',
            tabs:              [],
        } );

        const scrollWindow = ref( null );

        provide( 'tabsProvider', state );

        const storageKey = `vue-tabs-component.cache.${window.location.host}${window.location.pathname}`;

        const selectTab = ( selectedTabHash, event ) => {

            if( event && !props.options.useUrlFragment ) {
                event.preventDefault();
            }

            const selectedTab = findTab( selectedTabHash );

            if( !selectedTab ) {
                return;
            }

            if( selectedTab.isDisabled ) {
                event.preventDefault();
                return;
            }

            if( state.lastActiveTabHash === selectedTab.hash ) {
                context.emit( 'clicked', {tab: selectedTab} );
                return;
            }

            state.tabs.forEach( tab => {
                tab.isActive = (tab.hash === selectedTab.hash);
            } );

            context.emit( 'changed', {tab: selectedTab} );

            // On change, scroll the scroll windows to the top.
            if( scrollWindow.value !== null ) {
                scrollWindow.value.scrollTop = 0;
            }

            state.lastActiveTabHash = state.activeTabHash = selectedTab.hash;

            expiringStorage.set( storageKey, selectedTab.hash, props.cacheLifetime );
        };

        const findTab = ( hash ) => {
            return state.tabs.find( tab => tab.hash === hash );
        };

        onMounted( () => {
            if( !state.tabs.length ) {
                return;
            }

            window.addEventListener( 'hashchange', () => selectTab( window.location.hash ) );

            if( findTab( window.location.hash ) ) {
                selectTab( window.location.hash );
                return;
            }

            const previousSelectedTabHash = expiringStorage.get( storageKey );

            if( findTab( previousSelectedTabHash ) ) {
                selectTab( previousSelectedTabHash );
                return;
            }

            if( props.options.defaultTabHash && findTab( '#' + props.options.defaultTabHash ) ) {
                selectTab( '#' + props.options.defaultTabHash );
                return;
            }

            selectTab( state.tabs[0].hash );
        } );

        return {
            ...toRefs( state ),
            scrollWindow,
            selectTab,
            findTab,
        };
    },
};
</script>