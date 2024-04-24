# Get started with Nostr Connect

Nostr Connect ([NIP46](https://github.com/nostr-protocol/nips/blob/master/46.md)) is a remote key access protocol. 

If you need continuous access to users' Nostr keys, you have these options:
- store private keys in your app (please don't do that!)
- use `window.nostr` ([NIP07](https://github.com/nostr-protocol/nips/blob/master/07.md)), which is only available for web apps with a browser extension installed
- use [Amber](https://github.com/greenart7c3/Amber) or [Keystache](https://github.com/Resolvr-io/Keystache) or other platform-specific solution
- use *Nostr Connect* - a remote key access protocol that works on most platforms and is gaining wide adoption

This guide is for you if you want to implement Nostr Connect.

If you are looking for a more general *login with Nostr* guide, check out https://nostrlogin.org

## How to get started with Nostr Connect?

First things first, to test your Nostr Connect client you should get yourself a server first - try [nsec.app](https://nsec.app) or [nsecbunker.com](https://nsecbunker.com) or [Gossip](https://github.com/mikedilger/gossip) nostr client.

Now, if you're building a web app, the best place to start would be [nostr-login](https://github.com/nostrband/nostr-login) or [window.nostr.js](https://github.com/fiatjaf/window.nostr.js). 

These are drop-in scripts that provide the UI for users to log in with Nostr Connect, while exposing a `window.nostr` object for your app to work with. 

It super easy!

Just call `signEvent` and other methods on the `window.nostr` object and these libs will handle the rest.

Because there's actually a lot to handle...

## How to implement login with Nostr Connect?

If you just want your own Nostr Connect UI you should probably use [NDK](https://github.com/nostr-dev-kit/ndk/blob/master/ndk/src/signers/nip46/index.ts) or [nostr-tools](https://github.com/nbd-wtf/nostr-tools/blob/master/nip46.ts) to avoid implementing the full nip46 protocol yourself.

However, even with one of these libraries, the flow will be quite complex:
- ask the user for their `bunker URL` or `nip05` address
- show a *Loading* state to the user when they click *Login*, establishing a connection will take a while 
- if `nip05` address is entered, fetch the [nostr.json](https://github.com/nostr-protocol/nips/blob/master/46.md#nip-05-login-flow) info to get [remote user pubkey](https://github.com/nostr-protocol/nips/blob/master/46.md#terminology) and the `bunker relays` (that's `queryBunkerProfile` with `nostr-tools`, with `NDK` you just supply the `nip05` to the `NDKNip46Signer`)
- if bunker URL is entered, parse it properly to get the `remote user pubkey`, `bunker relays` and an optional `secret` token (example [#1](https://github.com/nbd-wtf/nostr-tools/blob/c12ddd3c538073ae8e43cf1199a750c0abb90531/nip46.ts#L31), [#2](https://github.com/verbiricha/habla.news/blob/54eb3c1edf28b41ac87086dc36b79ed241f8491e/src/components/nostr/Login.tsx#L146)) (that's `parseBunkerInput` with `nostr-tool`, with `NDK` you have to do that yourself)
- make sure you connect to the specified bunker relays (`NDK`)
- generate the [Local keypair](https://github.com/nostr-protocol/nips/blob/master/46.md#terminology), it will be used to sign nip46 requests to the `remote user pubkey`
- create a nip46 client and supply the `local keypair privkey` and other params to it, that's `BunkerSigner` with `nostr-tools` or `NDKNip46Signer` with `NDK` 
- subscribe to `auth_url` messages, that's `onauth` callback with `nostr-tools` or `authUrl` event with `NDKNip46Signer` - these are needed for users to confirm the actions you're requesting from their keys
- send a `connect` request, that's `BunkerSigner.connect` with `nostr-tools` or `NDKNip46Signer.blockUntilReady` with `NDK`
- if `auth_url` message arrives, you are supposed to open a popup browser tab with the provided URL
- show users a *Please confirm* button and only open the popup when they click it, opening without direct user action will get your popups blocked in many browsers
- the popup tabs are supposed to be closed automatically when user finishes interacting with them, you don't have to do that yourself
- when `connect` is finished, your connection is established and you're ready to make other methods calls
- save the `local keypair` and other params in persistent storage, reuse them next time your app starts

Wow, we made it! 

Well hopefully we didn't miss anything and it actually worked.

Here is what happens next:
- you send `sign_event` or other requests using the nip46 client
- `auth_url` messages might arrive if key storage app needs a confirmation from the user
- you should show a non-modal notification to the user saying *Please confirm*
- when user clicks, open the URL provided by `auth_url` message, opening it without a click will get your popup blocked and users irritated
- the popup will be auto-closed, no need to watch it

Cool, we're accessing those keys now!




