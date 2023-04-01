<script>
    import { afterUpdate, onMount } from 'svelte';
	import { selectedMessage, zappingMessage, zapsPerMessage } from './lib/store';
    import { chatData, chatAdapter } from './lib/store';
    import { nip19 } from 'nostr-tools';
    export let event;
    export let responses;
    export let websiteOwnerPubkey;

    let profiles = {};
    let profilePicture;
    let npub;
    let hovering;


    function selectMessage() {
        if ($selectedMessage === event.id) {
            $selectedMessage = null;
        } else {
            $selectedMessage = event.id;
        }
    }

    // delay-fetch responses
    onMount(() => {
        $chatAdapter.delayedSubscribe(
            {kinds: [1, 9735], '#e': [event.id]}
        , 'responses', 500)
    })

    const byWebsiteOwner = !!websiteOwnerPubkey === event.pubkey;
    // $: console.log('EVENT', profiles[event.pubkey], event)
    $: profiles = $chatData.profiles;
    $: displayName = profiles[event.pubkey] && profiles[event.pubkey].name || `[${event.pubkey.slice(0, 6)}]`;
    // $: nip05 = profiles[event.pubkey] && profiles[event.pubkey].nip05;
    $: zappingIt = $zappingMessage === event.id;
    $: {
        try {
            npub = nip19.npubEncode(event.pubkey);
        } catch (e) {
            npub = event.pubkey;
        }
    }


    $: profilePicture = profiles[event.pubkey] && profiles[event.pubkey].picture || `https://robohash.org/${event.pubkey.slice(0, 1)}.png?set=set1`;

    // const repliedIds = event.tags.filter(e => e[0] === 'e').map(e => e[1]);

    let timestamp = new Date(event.created_at * 1000);
</script>

<div class="inlagg"
    on:mouseenter={() => (hovering = true)}
    on:mouseleave={() => (hovering = false)}
>
    <div class="flex row">
        <div class="avatar">
            <a href={`https://iris.to/${npub}`}>
                <img src="{profilePicture}" class="
                    {byWebsiteOwner ? 'root' : ''}
                " alt="" />
            </a>
        </div>

        <div class="details">
            <div>
                <strong>{displayName}</strong>
                <small class="">{timestamp.toLocaleString()}</small>
            </div>
            <p class="texten" style="white-space: pre-line;">{event.content.replace(/(https?:\/\/[^\s]+)/g, '[Länk dold]')}</p>
            <div class="flex row center">
                <a href={`https://iris.to/${nip19.noteEncode(event.id)}`}>Permalänk</a>
            </div>
        </div>
    </div>
</div>

{#if responses[event.id].length > 0}
    <div class="pl-5 border-l border-l-gray-400 flex flex-col gap-4">
        {#each responses[event.id] as response}
            <svelte:self {websiteOwnerPubkey} event={response} {responses} />
        {/each}
    </div>
{/if}

<style>

</style>