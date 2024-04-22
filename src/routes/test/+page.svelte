<script lang="ts" type="module">
	import { redirectToAuthCodeFlow, getAccessToken } from '$lib/scripts/test';
	import { onMount } from 'svelte';

    let accessToken: any = null

	async function start_profile_fetch(code: string | null, clientId: string) {
		if (!code) {
			redirectToAuthCodeFlow(clientId);
		} else {
			accessToken = await getAccessToken(clientId, code);
			const profile = await fetchProfile(accessToken);
			populateUI(profile);
		}
	}

	async function start_playlist_fetch(playlistId: string) {;
		if (!accessToken) {
            console.error('Invalid access token')
            return
		}

        const playlistData = await fetchPlaylist(playlistId, accessToken)
        logPlaylist(playlistData)
	}

	async function logPlaylist(playlistData: any) {
		console.log(playlistData.tracks);
	}

	async function fetchProfile(token: string): Promise<any> {
		const result = await fetch('https://api.spotify.com/v1/me', {
			method: 'GET',
			headers: { Authorization: `Bearer ${token}` }
		});

		return await result.json();
	}

	async function fetchPlaylist(playlistId: string, token: string): Promise<any> {
		const result = await fetch(`https://api.spotify.com/v1/playlists/${playlistId}`, {
			method: 'GET',
			headers: { Authorization: `Bearer ${token}` }
		});

		return await result.json();
	}

	function populateUI(profile: any) {
		document.getElementById('displayName')!.innerText = profile.display_name;
		if (profile.images[0]) {
			const profileImage = new Image(200, 200);
			profileImage.src = profile.images[0].url;
			document.getElementById('avatar')!.appendChild(profileImage);
		}
	}

	onMount(() => {
		const clientId = '1745fe5ef35f425d804e36993d363520';
		const params = new URLSearchParams(window.location.search);
		const code = params.get('code');

		start_profile_fetch(code, clientId);
	});
</script>

<section id="profile">
	<h1>Logged in as <span id="displayName"></span></h1>
	<span id="avatar"></span>

	<button on:click={() => {
        start_playlist_fetch('26KIHuyhLnZyJjLeLScoqB')
    }}>Fetch Playlist</button>
</section>
