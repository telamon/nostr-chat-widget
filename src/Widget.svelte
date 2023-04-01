<script>
    import Container from './Container.svelte';

    export let websiteOwnerPubkey;
    export let chatType;
    export let chatTags;
    export let chatReferenceTags;
    export let relays;

    let showChat = true;
    let dismissedIntro = true;
    let minimizeChat = false;

    function toggleChat() {
        if (showChat) {
            minimizeChat = !minimizeChat;
        } else {
            showChat = !showChat;
        }
    }

    function dismissIntro() {
        dismissedIntro = true;
    }
</script>
<guest-book>
    <Container
        {websiteOwnerPubkey}
        chatConfiguration={{
            chatType,
            chatTags,
            chatReferenceTags,
        }}
        {relays}
    />
</guest-book>