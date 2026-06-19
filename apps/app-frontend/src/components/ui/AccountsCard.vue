<template>
	<div
		v-if="accounts.length === 0"
		class="flex flex-col gap-3 bg-button-bg border border-solid border-surface-5 rounded-xl p-3 mt-2"
	>
		<span>{{ formatMessage(messages.notSignedIn) }}</span>
		<ButtonStyled color="brand">
			<button color="primary" :disabled="loginDisabled" @click="login()">
				<MicrosoftIcon v-if="!loginDisabled" />
				<SpinnerIcon v-else class="animate-spin" />
				{{ formatMessage(messages.signInToMinecraft) }}
			</button>
		</ButtonStyled>
		<!-- BEGIN: This code block modified by AstralRinth -->
		<ButtonStyled class="w-full">
			<OverflowMenu class="w-full justify-between text-left" :options="additionalAccountOptions">
				<span class="inline-flex items-center gap-2">
					<PlusIcon />
					{{ formatMessage(messages.addAccount) }}
				</span>
				<DropdownIcon class="shrink-0" />
				<template #add_offline_account>
					<OfflineIcon />
					{{ formatMessage(messages.addOfflineAccount) }}
				</template>
				<template #add_elyby_account>
					<ElyByIcon v-if="!elyByLoginDisabled" />
					<SpinnerIcon v-else class="animate-spin" />
					{{ formatMessage(messages.addElyByAccount) }}
				</template>
			</OverflowMenu>
		</ButtonStyled>
		<!-- END: This code block modified by AstralRinth -->
	</div>
	<Accordion
		v-else
		class="w-full mt-2 bg-button-bg border border-solid border-surface-5 rounded-xl overflow-clip"
		button-class="button-base w-full bg-transparent px-3 py-2 border-0 cursor-pointer"
		:open-by-default="false"
	>
		<template #title>
			<div class="flex gap-2 w-full min-w-0">
				<Avatar
					size="36px"
					:src="
						selectedAccount
							? avatarUrl
							: 'https://launcher-files.modrinth.com/assets/steve_head.png'
					"
				/>
				<div class="flex flex-col items-start w-full min-w-0">
					<span class="truncate w-full text-left">
						<span class="inline-flex items-center gap-1 min-w-0">
							<component
								:is="getAccountType(selectedAccount)"
								v-if="selectedAccount && getAccountType(selectedAccount)"
								class="vector-icon shrink-0"
							/>
							<span class="truncate">
								{{
									selectedAccount
										? selectedAccount.profile.name
										: formatMessage(messages.selectAccount)
								}}
							</span>
						</span>
					</span>
					<span class="text-secondary text-xs">{{ formatMessage(messages.minecraftAccount) }}</span>
				</div>
			</div>
		</template>
		<div class="bg-button-bg pt-1 pb-2 border border-solid border-surface-5">
			<template v-if="accounts.length > 0">
				<div v-for="account in accounts" :key="account.profile.id" class="flex gap-1 items-center">
					<button
						class="flex items-center flex-shrink flex-grow overflow-clip gap-2 p-2 border-0 bg-transparent cursor-pointer button-base min-w-0"
						@click="setAccount(account)"
					>
						<RadioButtonCheckedIcon
							v-if="selectedAccount && selectedAccount.profile.id === account.profile.id"
							class="w-5 h-5 text-brand shrink-0"
						/>
						<RadioButtonIcon v-else class="w-5 h-5 text-secondary shrink-0" />
						<Avatar :src="getAccountAvatarUrl(account)" size="24px" />
						<p
							class="m-0 truncate min-w-0 inline-flex items-center gap-1"
							:class="
								selectedAccount && selectedAccount.profile.id === account.profile.id
									? 'text-contrast font-semibold'
									: 'text-primary'
							"
						>
							<component
								:is="getAccountType(account)"
								v-if="getAccountType(account)"
								class="vector-icon shrink-0"
							/>
							<span class="truncate">{{ account.profile.name }}</span>
						</p>
					</button>
					<ButtonStyled circular color="red" color-fill="none" hover-color-fill="background">
						<button
							v-tooltip="formatMessage(messages.removeAccount)"
							class="mr-2"
							@click="logout(account.profile.id)"
						>
							<TrashIcon />
						</button>
					</ButtonStyled>
				</div>
			</template>
			<div class="flex flex-col gap-2 px-2 pt-2">
				<ButtonStyled class="w-full" color="brand">
					<button :disabled="loginDisabled" @click="login()">
						<MicrosoftIcon v-if="!loginDisabled" />
						<SpinnerIcon v-else class="animate-spin" />
						{{ formatMessage(messages.addMicrosoftAccount) }}
					</button>
				</ButtonStyled>
				<ButtonStyled class="w-full">
					<OverflowMenu class="w-full justify-between text-left" :options="additionalAccountOptions">
						<span class="inline-flex items-center gap-2">
							<PlusIcon />
							{{ formatMessage(messages.addAccount) }}
						</span>
						<DropdownIcon class="shrink-0" />
						<template #add_offline_account>
							<OfflineIcon />
							{{ formatMessage(messages.addOfflineAccount) }}
						</template>
						<template #add_elyby_account>
							<ElyByIcon v-if="!elyByLoginDisabled" />
							<SpinnerIcon v-else class="animate-spin" />
							{{ formatMessage(messages.addElyByAccount) }}
						</template>
					</OverflowMenu>
				</ButtonStyled>
			</div>
		</div>
	</Accordion>
	<ModalWrapper ref="addElyByModal" class="modal" header="Authenticate with Ely.by">
		<ModalWrapper
			ref="requestElyByTwoFactorCodeModal"
			class="modal"
			header="Ely.by requested 2FA code for authentication"
		>
			<div class="flex flex-col gap-4 px-6 py-5">
				<label class="label form-label">Enter your 2FA code</label>
				<input
					v-model="elyByTwoFactorCode"
					type="text"
					placeholder="Your 2FA code here..."
					class="input soft-input"
				/>
				<div class="mt-6 ml-auto">
					<Button color="primary" :disabled="elyByLoginDisabled" @click="addElyByProfile()">
						Continue
					</Button>
				</div>
			</div>
		</ModalWrapper>
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="label form-label">Enter your player name or email (preferred)</label>
			<input
				v-model="elyByLoginValue"
				type="text"
				placeholder="Your player name or email here..."
				class="input soft-input"
			/>
			<label class="label form-label">Enter your password</label>
			<input
				v-model="elyByPassword"
				type="password"
				placeholder="Your password here..."
				class="input soft-input"
			/>
			<div class="mt-6 ml-auto">
				<Button color="primary" :disabled="elyByLoginDisabled" @click="addElyByProfile()">
					Login
				</Button>
			</div>
		</div>
	</ModalWrapper>
	<ModalWrapper ref="addOfflineModal" class="modal" header="Add new offline account">
		<div class="flex flex-col gap-4 px-6 py-5">
			<label class="label form-label">Enter your player name</label>
			<input
				v-model="offlinePlayerName"
				type="text"
				placeholder="Your player name here..."
				class="input soft-input"
			/>
			<div class="mt-6 ml-auto">
				<Button color="primary" @click="addOfflineProfile()">Login</Button>
			</div>
		</div>
	</ModalWrapper>
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
				<Button color="primary" @click="retryAddElyByProfile">Try again</Button>
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
				<Button color="primary" @click="retryAddElyByProfile">Try again</Button>
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
				<Button color="primary" @click="retryAddOfflineProfile">Try again</Button>
			</div>
		</div>
	</ModalWrapper>
	<ModalWrapper ref="unexpectedErrorModal" class="modal" header="Unexpected error occurred">
		<div class="modal-body">
			<label class="label">An unexpected error has occurred. Please try again later.</label>
		</div>
	</ModalWrapper>
</template>

<script setup lang="ts">
import ModalWrapper from '@/components/ui/modal/ModalWrapper.vue'
import { trackEvent } from '@/helpers/analytics'
import {
	elyby_auth_authenticate,
	elyby_login,
	get_default_user,
	login as login_flow,
	offline_login,
	remove_user,
	set_default_user,
	users,
} from '@/helpers/auth'
import { process_listener } from '@/helpers/events'
import { getPlayerHeadUrl } from '@/helpers/rendering/batch-skin-renderer.ts'
import type { Skin } from '@/helpers/skins'
import { get_available_skins } from '@/helpers/skins'
import { handleSevereError } from '@/store/error.js'
import {
	DropdownIcon,
	ElyByIcon,
	MicrosoftIcon,
	OfflineIcon,
	RadioButtonCheckedIcon,
	RadioButtonIcon,
	SpinnerIcon,
	TrashIcon,
} from '@modrinth/assets'
import {
	Accordion,
	Avatar,
	Button,
	ButtonStyled,
	defineMessages,
	injectNotificationManager,
	OverflowMenu,
	useVIntl,
} from '@modrinth/ui'
import type { Ref } from 'vue'
import { computed, onUnmounted, ref } from 'vue'

const { formatMessage } = useVIntl()
const { handleError } = injectNotificationManager()

const emit = defineEmits<{
	change: []
}>()

type MinecraftCredential = {
	account_type?: 'microsoft' | 'offline' | 'elyby' | string
	profile: {
		id: string
		name: string
	}
}

type ModalHandle = {
	hide: () => void
	show: () => void
}

const clientToken = 'astralrinth'
const minOfflinePlayerNameLength = 3
const maxOfflinePlayerNameLength = 20
const nameExp = 'a-zA-Z0-9_'
const nameRegex = new RegExp(`^[${nameExp}]+$`)

const accounts: Ref<MinecraftCredential[]> = ref([])
const loginDisabled = ref(false)
const elyByLoginDisabled = ref(false)
const defaultUser = ref<string | undefined>()
const equippedSkin = ref<Skin | null>(null)
const headUrlCache = ref(new Map<string, string>())

const addOfflineModal = ref<ModalHandle | null>(null)
const addElyByModal = ref<ModalHandle | null>(null)
const requestElyByTwoFactorCodeModal = ref<ModalHandle | null>(null)
const authenticationElyByErrorModal = ref<ModalHandle | null>(null)
const inputElyByErrorModal = ref<ModalHandle | null>(null)
const inputOfflineErrorModal = ref<ModalHandle | null>(null)
const unexpectedErrorModal = ref<ModalHandle | null>(null)

const offlinePlayerName = ref('')
const elyByLoginValue = ref('')
const elyByPassword = ref('')
const elyByTwoFactorCode = ref('')

function getAccountType(account?: MinecraftCredential) {
	switch (account?.account_type) {
		case 'microsoft':
			return MicrosoftIcon
		case 'offline':
			return OfflineIcon
		case 'elyby':
			return ElyByIcon
		default:
			return null
	}
}

function showOfflineLoginModal() {
	addOfflineModal.value?.show()
}

function showElyByLoginModal() {
	addElyByModal.value?.show()
}

const additionalAccountOptions = computed(() => [
	{
		id: 'add_offline_account',
		action: showOfflineLoginModal,
	},
	{
		id: 'add_elyby_account',
		action: showElyByLoginModal,
		disabled: elyByLoginDisabled.value,
	},
])

function retryAddOfflineProfile() {
	inputOfflineErrorModal.value?.hide()
	clearOfflineFields()
	showOfflineLoginModal()
}

function retryAddElyByProfile() {
	authenticationElyByErrorModal.value?.hide()
	inputElyByErrorModal.value?.hide()
	elyByLoginDisabled.value = false
	clearElyByFields()
	showElyByLoginModal()
}

function clearElyByFields() {
	elyByLoginValue.value = ''
	elyByPassword.value = ''
	elyByTwoFactorCode.value = ''
}

function clearOfflineFields() {
	offlinePlayerName.value = ''
}

async function addOfflineProfile() {
	const name = offlinePlayerName.value.trim()
	const isValidName =
		nameRegex.test(name) &&
		name.length >= minOfflinePlayerNameLength &&
		name.length <= maxOfflinePlayerNameLength

	if (!isValidName) {
		addOfflineModal.value?.hide()
		inputOfflineErrorModal.value?.show()
		clearOfflineFields()
		return
	}

	try {
		const result = await offline_login(name)
		addOfflineModal.value?.hide()

		if (result) {
			await setAccount(result)
			await refreshValues()
		} else {
			unexpectedErrorModal.value?.show()
		}
	} catch (error) {
		handleError(error)
		unexpectedErrorModal.value?.show()
	} finally {
		clearOfflineFields()
	}
}

async function addElyByProfile() {
	elyByLoginDisabled.value = true

	if (!elyByLoginValue.value || !elyByPassword.value) {
		addElyByModal.value?.hide()
		inputElyByErrorModal.value?.show()
		clearElyByFields()
		elyByLoginDisabled.value = false
		return
	}

	const login = elyByLoginValue.value.trim()
	let password = elyByPassword.value.trim()
	const twoFactorCode = elyByTwoFactorCode.value.trim()

	if (password && twoFactorCode) {
		password = `${password}:${twoFactorCode}`
	}

	try {
		const rawResult = await elyby_auth_authenticate(login, password, clientToken)
		const jsonData = JSON.parse(rawResult)

		if (!jsonData.accessToken) {
			if (
				jsonData.error === 'ForbiddenOperationException' &&
				jsonData.errorMessage?.includes('two factor')
			) {
				requestElyByTwoFactorCodeModal.value?.show()
				return
			}

			addElyByModal.value?.hide()
			requestElyByTwoFactorCodeModal.value?.hide()
			authenticationElyByErrorModal.value?.show()
			return
		}

		const accessToken = jsonData.accessToken
		const selectedProfileId = convertRawStringToUUIDv4(jsonData.selectedProfile.id)
		const selectedProfileName = jsonData.selectedProfile.name
		const result = await elyby_login(selectedProfileId, selectedProfileName, accessToken)

		addElyByModal.value?.hide()
		requestElyByTwoFactorCodeModal.value?.hide()
		clearElyByFields()

		await setAccount(result)
		await refreshValues()
	} catch (error) {
		handleError(error)
		unexpectedErrorModal.value?.show()
	} finally {
		elyByLoginDisabled.value = false
	}
}

function convertRawStringToUUIDv4(rawId: string) {
	if (rawId.length !== 32) {
		console.warn('Invalid UUID string:', rawId)
		return rawId
	}

	return `${rawId.slice(0, 8)}-${rawId.slice(8, 12)}-${rawId.slice(12, 16)}-${rawId.slice(16, 20)}-${rawId.slice(20)}`
}

async function refreshValues() {
	defaultUser.value = await get_default_user().catch(handleError)
	const userList = await users().catch(handleError)
	accounts.value = Array.isArray(userList) ? [...userList] : []
	accounts.value.sort((a, b) => (a.profile?.name ?? '').localeCompare(b.profile?.name ?? ''))

	try {
		const skins = await get_available_skins()
		equippedSkin.value = skins.find((skin) => skin.is_equipped) ?? null

		if (equippedSkin.value) {
			try {
				const headUrl = await getPlayerHeadUrl(equippedSkin.value)
				headUrlCache.value = new Map(headUrlCache.value).set(
					equippedSkin.value.texture_key,
					headUrl,
				)
			} catch (error) {
				console.warn('Failed to get head render for equipped skin:', error)
			}
		}
	} catch {
		equippedSkin.value = null
	}
}

async function setEquippedSkin(skin: Skin) {
	equippedSkin.value = skin

	try {
		const headUrl = await getPlayerHeadUrl(skin)
		headUrlCache.value = new Map(headUrlCache.value).set(skin.texture_key, headUrl)
	} catch (error) {
		console.warn('Failed to get head render for equipped skin:', error)
	}
}

function setLoginDisabled(value: boolean) {
	loginDisabled.value = value
}

defineExpose({
	refreshValues,
	setEquippedSkin,
	setLoginDisabled,
	loginDisabled,
})

await refreshValues()

const selectedAccount = computed(() =>
	accounts.value.find((account) => account.profile.id === defaultUser.value),
)

const avatarUrl = computed(() => {
	if (equippedSkin.value?.texture_key) {
		const cachedUrl = headUrlCache.value.get(equippedSkin.value.texture_key)
		if (cachedUrl) {
			return cachedUrl
		}

		return `https://mc-heads.net/avatar/${equippedSkin.value.texture_key}/128`
	}

	if (selectedAccount.value?.profile?.id) {
		return `https://mc-heads.net/avatar/${selectedAccount.value.profile.id}/128`
	}

	return 'https://launcher-files.modrinth.com/assets/steve_head.png'
})

function getAccountAvatarUrl(account: MinecraftCredential) {
	if (
		account.profile.id === selectedAccount.value?.profile?.id &&
		equippedSkin.value?.texture_key
	) {
		const cachedUrl = headUrlCache.value.get(equippedSkin.value.texture_key)
		if (cachedUrl) {
			return cachedUrl
		}
	}

	return `https://mc-heads.net/avatar/${account.profile.id}/128`
}

async function setAccount(account: MinecraftCredential) {
	defaultUser.value = account.profile.id
	await set_default_user(account.profile.id).catch(handleError)
	await refreshValues()
	emit('change')
}

async function login() {
	loginDisabled.value = true
	const loggedIn = await login_flow().catch(handleSevereError)

	if (loggedIn) {
		await setAccount(loggedIn)
	}

	trackEvent('AccountLogIn')
	loginDisabled.value = false
}

async function logout(id: string) {
	await remove_user(id).catch(handleError)
	await refreshValues()

	if (!selectedAccount.value && accounts.value.length > 0) {
		await setAccount(accounts.value[0])
	} else {
		emit('change')
	}

	trackEvent('AccountLogOut')
}

const unlisten = await process_listener(async (e) => {
	if (e.event === 'launched') {
		await refreshValues()
	}
})

onUnmounted(() => {
	unlisten()
})

const messages = defineMessages({
	notSignedIn: {
		id: 'minecraft-account.not-signed-in',
		defaultMessage: 'Not signed in',
	},
	addAccount: {
		id: 'minecraft-account.add-account',
		defaultMessage: 'Add account',
	},
	addMicrosoftAccount: {
		id: 'minecraft-account.add-microsoft-account',
		defaultMessage: 'Add Microsoft account',
	},
	addOfflineAccount: {
		id: 'astralrinth.app.minecraft-account.add-offline-account',
		defaultMessage: 'Add offline account',
	},
	addElyByAccount: {
		id: 'astralrinth.app.minecraft-account.add-elyby-account',
		defaultMessage: 'Add Ely.by account',
	},
	removeAccount: {
		id: 'minecraft-account.remove-account',
		defaultMessage: 'Remove account',
	},
	selectAccount: {
		id: 'minecraft-account.select-account',
		defaultMessage: 'Select account',
	},
	minecraftAccount: {
		id: 'minecraft-account.label',
		defaultMessage: 'Minecraft account',
	},
	signInToMinecraft: {
		id: 'minecraft-account.sign-in',
		defaultMessage: 'Sign in to Minecraft',
	},
})
</script>

<style scoped lang="scss">
@import '../../../../../packages/assets/styles/astralrinth/soft-inputs.scss';

.vector-icon {
	width: 0.875rem;
	height: 0.875rem;
}

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
