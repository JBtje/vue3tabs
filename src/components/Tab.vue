<template>
    <section v-if="isActive || isShown"
             v-show="isActive"
             :aria-hidden="!isActive"
             :class="panelClass"
             :id="(link ? '' : 'tab-') + computedId"
             role="tabpanel"
             ref="tab"
    >
        <slot/>
    </section>
</template>

<script>
import {inject, onBeforeMount, ref, watch} from 'vue';

export default {
    name:  'Tab',
    props: {
        id:         {
            type:    String,
            default: null,
        },
        isDisabled: {
            type:    Boolean,
            default: false,
        },
        name:       {
            default:  '',
            required: true,
        },
        panelClass: {
            type:    String,
            default: 'tabs-component-panel',
        },
        prefix:     {
            default: '',
        },
        suffix:     {
            default: '',
        },
        link:       {
            type:    Boolean,
            default: false,
        },
    },

    setup( props ) {
        const isActive = ref( false );
        const isShown  = ref( false );

        const tabsProvider = inject( 'tabsProvider' );

        // Wrap prefix, name and suffix in a function.
        const prefix     = typeof props.prefix === 'function' ? props.prefix : () => props.prefix;
        const header     = typeof props.name === 'function' ? props.name : () => props.name;
        const suffix     = typeof props.suffix === 'function' ? props.suffix : () => props.suffix;
        const computedId = props.id ? props.id : header().toLowerCase().replace( / /g, '-' );
        const hash       = '#' + (!props.isDisabled ? computedId : '');

        watch(
            () => tabsProvider.activeTabHash,
            () => {
                isActive.value = hash === tabsProvider.activeTabHash;
            },
        );

        watch(
            () => isActive.value,
            () => {
                // Once the content has mounted, keep it mounted.
                isShown.value = isShown.value || isActive.value;
            },
        );

        onBeforeMount( () => {
            tabsProvider.tabs.push( {
                prefix:     prefix,
                header:     header,
                suffix:     suffix,
                isDisabled: props.isDisabled,
                hash:       hash,
                index:      tabsProvider.tabs.length,
            } );
        } );

        return {
            prefix,
            header,
            suffix,
            computedId,
            hash,
            isActive,
            isShown,
        };
    },
};
</script>