<template>
    <section v-if="isActive || isShown"
             v-show="isActive"
             :aria-hidden="!isActive"
             :class="panelClass"
             :id="'tab-' + computedId"
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
            type:     String,
            required: true,
        },
        panelClass: {
            type:    String,
            default: 'tabs-component-panel',
        },
        prefix:     {
            type:    String,
            default: '',
        },
        suffix:     {
            type:    String,
            default: '',
        },
    },

    setup( props ) {
        const isActive = ref( false );
        const isShown  = ref( false );

        const tabsProvider = inject( 'tabsProvider' );

        const header     = props.prefix + props.name + props.suffix;
        const computedId = props.id ? props.id : props.name.toLowerCase().replace( / /g, '-' );
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
                name:       props.name,
                header:     header,
                isDisabled: props.isDisabled,
                hash:       hash,
                index:      tabsProvider.tabs.length,
            } );
        } );

        return {
            header,
            computedId,
            hash,
            isActive,
        };
    },
};
</script>