<script setup lang="ts">
import { Button } from '@modrinth/ui'
import { ref } from 'vue'

import ModalWrapper from '@/components/ui/modal/ModalWrapper.vue'

type ModalHandle = {
	hide: () => void
	show: () => void
}

defineProps<{
	maxOfflinePlayerNameLength: number
	minOfflinePlayerNameLength: number
	nameExp: string
}>()

const emit = defineEmits<{
	(event: 'retry-elyby'): void
	(event: 'retry-offline'): void
}>()

const authenticationElyByErrorModal = ref<ModalHandle | null>(null)
const inputElyByErrorModal = ref<ModalHandle | null>(null)
const inputOfflineErrorModal = ref<ModalHandle | null>(null)
const unexpectedErrorModal = ref<ModalHandle | null>(null)

defineExpose({
	hideAuthenticationElyByError: () => authenticationElyByErrorModal.value?.hide(),
	hideInputElyByError: () => inputElyByErrorModal.value?.hide(),
	hideInputOfflineError: () => inputOfflineErrorModal.value?.hide(),
	showAuthenticationElyByError: () => authenticationElyByErrorModal.value?.show(),
	showInputElyByError: () => inputElyByErrorModal.value?.show(),
	showInputOfflineError: () => inputOfflineErrorModal.value?.show(),
	showUnexpectedError: () => unexpectedErrorModal.value?.show(),
})
</script>

<template>
	<ModalWrapper
		ref="authenticationElyByErrorModal"
		class="modal"
		header="Error while proceeding authentication event with Ely.by"
	>
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="text-base font-medium text-red-700">
				An error occurred while logging in.
			</label>
			<div class="mt-6 ml-auto">
				<Button color="primary" @click="emit('retry-elyby')">Try again</Button>
			</div>
		</div>
	</ModalWrapper>
	<ModalWrapper
		ref="inputElyByErrorModal"
		class="modal"
		header="Error while proceeding input event with Ely.by"
	>
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="text-base font-medium text-red-700">
				An error occurred while adding the Ely.by account. Please follow the instructions below.
			</label>
			<ul class="list-disc list-inside text-sm space-y-1">
				<li>Check that you have entered the correct player name or email.</li>
				<li>Check that you have entered the correct password.</li>
			</ul>
			<div class="mt-6 ml-auto">
				<Button color="primary" @click="emit('retry-elyby')">Try again</Button>
			</div>
		</div>
	</ModalWrapper>
	<ModalWrapper
		ref="inputOfflineErrorModal"
		class="modal"
		header="Error while proceeding input event with offline account"
	>
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="text-base font-medium text-red-700">
				An error occurred while adding the offline account. Please follow the instructions below.
			</label>
			<ul class="list-disc list-inside text-sm space-y-1">
				<li>Check that you have entered the correct player name.</li>
				<li>
					Player name must be at least {{ minOfflinePlayerNameLength }} characters long and no more
					than {{ maxOfflinePlayerNameLength }} characters.
				</li>
				<li>Make sure your name meets the format requirement `{{ nameExp }}`</li>
			</ul>
			<div class="mt-6 ml-auto">
				<Button color="primary" @click="emit('retry-offline')">Try again</Button>
			</div>
		</div>
	</ModalWrapper>
	<ModalWrapper ref="unexpectedErrorModal" class="modal" header="Unexpected error occurred">
		<div class="modal-body">
			<label class="label">An unexpected error has occurred. Please try again later.</label>
		</div>
	</ModalWrapper>
</template>

<style scoped lang="scss">
.modal {
	position: absolute;
}

.modal-body {
	display: flex;
	flex-direction: row;
	gap: var(--gap-lg);
	align-items: center;
	padding: var(--gap-xl);
}
</style>
