<script setup lang="ts">
import { Button } from '@modrinth/ui'
import { ref } from 'vue'

import ModalWrapper from '@/components/ui/modal/ModalWrapper.vue'

type ModalHandle = {
	hide: () => void
	show: () => void
}

const props = defineProps<{
	elyByLoginDisabled: boolean
	elyByLoginValue: string
	elyByPassword: string
	elyByTwoFactorCode: string
	offlineLoginDisabled: boolean
	offlinePlayerName: string
}>()

const emit = defineEmits<{
	(event: 'submit-elyby'): void
	(event: 'submit-offline'): void
	(event: 'update:elyByLoginValue', value: string): void
	(event: 'update:elyByPassword', value: string): void
	(event: 'update:elyByTwoFactorCode', value: string): void
	(event: 'update:offlinePlayerName', value: string): void
}>()

const addOfflineModal = ref<ModalHandle | null>(null)
const addElyByModal = ref<ModalHandle | null>(null)
const requestElyByTwoFactorCodeModal = ref<ModalHandle | null>(null)

defineExpose({
	hideElyBy: () => addElyByModal.value?.hide(),
	hideElyByTwoFactor: () => requestElyByTwoFactorCodeModal.value?.hide(),
	hideOffline: () => addOfflineModal.value?.hide(),
	showElyBy: () => addElyByModal.value?.show(),
	showElyByTwoFactor: () => requestElyByTwoFactorCodeModal.value?.show(),
	showOffline: () => addOfflineModal.value?.show(),
})
</script>

<template>
	<ModalWrapper ref="addElyByModal" class="modal" header="Authenticate with Ely.by">
		<ModalWrapper
			ref="requestElyByTwoFactorCodeModal"
			class="modal"
			header="Ely.by requested 2FA code for authentication"
		>
			<div class="flex flex-col gap-4 px-6 py-5">
				<label class="label form-label">Enter your 2FA code</label>
				<input
					:value="props.elyByTwoFactorCode"
					type="text"
					placeholder="Your 2FA code here..."
					class="input soft-input"
					@input="
						emit('update:elyByTwoFactorCode', ($event.target as HTMLInputElement).value)
					"
				/>
				<div class="mt-6 ml-auto">
					<Button color="primary" :disabled="props.elyByLoginDisabled" @click="emit('submit-elyby')">
						Continue
					</Button>
				</div>
			</div>
		</ModalWrapper>
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="label form-label">Enter your player name or email (preferred)</label>
			<input
				:value="props.elyByLoginValue"
				type="text"
				placeholder="Your player name or email here..."
				class="input soft-input"
				@input="emit('update:elyByLoginValue', ($event.target as HTMLInputElement).value)"
			/>
			<label class="label form-label">Enter your password</label>
			<input
				:value="props.elyByPassword"
				type="password"
				placeholder="Your password here..."
				class="input soft-input"
				@input="emit('update:elyByPassword', ($event.target as HTMLInputElement).value)"
			/>
			<div class="mt-6 ml-auto">
				<Button color="primary" :disabled="props.elyByLoginDisabled" @click="emit('submit-elyby')">
					Login
				</Button>
			</div>
		</div>
	</ModalWrapper>
	<ModalWrapper ref="addOfflineModal" class="modal" header="Add new offline account">
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="label form-label">Enter your player name</label>
			<input
				:value="props.offlinePlayerName"
				type="text"
				placeholder="Your player name here..."
				class="input soft-input"
				@input="emit('update:offlinePlayerName', ($event.target as HTMLInputElement).value)"
			/>
			<div class="mt-6 ml-auto">
				<Button color="primary" :disabled="props.offlineLoginDisabled" @click="emit('submit-offline')">
					Login
				</Button>
			</div>
		</div>
	</ModalWrapper>
</template>

<style scoped lang="scss">
@import '../../../../../../../../packages/assets/styles/astralrinth/soft-inputs.scss';

.modal {
	position: absolute;
}
</style>
