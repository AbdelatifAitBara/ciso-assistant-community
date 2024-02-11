<script lang="ts">
	import { page } from '$app/stores';
	import { popup } from '@skeletonlabs/skeleton';
	import type { ModalSettings, PopupSettings } from '@skeletonlabs/skeleton';
	import { getModalStore } from '@skeletonlabs/skeleton';

	const modalStore = getModalStore();

	const popupUser: PopupSettings = {
		event: 'click',
		target: 'popupUser',
		placement: 'top'
	};

	async function modalBuildInfo() {
		const res = await fetch('/api/build');
		const { version, build } = await res.json();
		const modal: ModalSettings = {
			type: 'component',
			component: 'displayJSONModal',
			title: 'About CISO Assistant',
			body: JSON.stringify({ version, build })
		};
		modalStore.trigger(modal);
	}
</script>

<div class="border-t pt-2.5">
	<div class="flex flex-row items-center justify-between">
		<div class="flex flex-col" data-testid="sidebar-user-account-display">
			{#if $page.data.user}
				<span class="text-gray-900 text-sm whitespace-nowrap overflow-hidden truncate">
					{$page.data.user.first_name}
					{$page.data.user.last_name}
				</span>
				<span
					class="font-normal text-xs text-gray-600 mr-2 whitespace-nowrap overflow-hidden truncate w-full"
				>
					{$page.data.user.email}
				</span>
			{/if}
		</div>
		<button class="btn bg-initial" data-testid="sidebar-more-btn" use:popup={popupUser}
			><i class="fa-solid fa-ellipsis-vertical" /></button
		>
		<div
			class="card whitespace-nowrap bg-white py-2 w-fit shadow-lg space-y-1"
			data-testid="sidebar-more-panel"
			data-popup="popupUser"
		>
			<a
				href="/profile"
				class="unstyled cursor-pointer flex items-center gap-2 w-full px-4 py-2.5 text-left text-sm hover:bg-gray-100 disabled:text-gray-500 text-gray-800"
				data-testid="profile-button"><i class="fa-solid fa-address-card mr-2" />My profile</a
			>
			<button
				on:click={modalBuildInfo}
				class="cursor-pointer flex items-center gap-2 w-full px-4 py-2.5 text-left text-sm hover:bg-gray-100 disabled:text-gray-500 text-gray-800"
				data-testid="about-button"
				><i class="fa-solid fa-circle-info mr-2" />About CISO Assistant</button
			>
			<form action="/logout" method="POST">
				<button class="w-full" type="submit" data-testid="logout-button">
					<span
						class="flex items-center gap-2 w-full px-4 py-2.5 text-left text-sm hover:bg-gray-100 disabled:text-gray-500 text-gray-800"
						><i class="fa-solid fa-right-from-bracket mr-2" />Log out</span
					>
				</button>
			</form>
		</div>
	</div>
</div>