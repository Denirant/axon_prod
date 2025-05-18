<script lang="ts">
    import TagInput from './Tags/TagInput.svelte';
    import TagList from './Tags/TagList.svelte';
    import { getContext, createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    const i18n = getContext('i18n');

    export let tags = [];
</script>

<div class="flex flex-col gap-1.5 items-center">

	<TagInput
        label={$i18n.t('Add Tags')}
        on:add={(e) => {
            dispatch('add', e.detail);
        }}
    />

    <!-- <TagList
        {tags}
        on:delete={(e) => {
            dispatch('delete', e.detail);
        }}
    /> -->
</div>