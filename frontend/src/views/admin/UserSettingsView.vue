<template>
	<div class="admin-user-settings">
		<div class="admin-user-settings__header">
			<RouterLink :to="{name: 'admin.users'}">
				{{ $t('admin.users.backToUsers') }}
			</RouterLink>
			<h2
				v-if="data"
				class="title is-4"
			>
				{{ $t('admin.users.settingsTitle', {username: data.user?.username}) }}
			</h2>
		</div>

		<p v-if="isPending">
			{{ $t('misc.loading') }}
		</p>
		<!-- The form seeds its draft once, so it may only mount with the settings loaded. -->
		<GeneralSettingsForm
			v-else-if="data"
			:key="userId"
			:initial-settings="data.settings"
			:is-external-user="!!data.user?.auth_provider"
			:auth-provider="data.user?.auth_provider"
			:save="save"
			:own-account="false"
		/>
	</div>
</template>

<script setup lang="ts">
import {computed} from 'vue'
import {useI18n} from 'vue-i18n'
import {useQuery} from '@tanstack/vue-query'

import GeneralSettingsForm from '@/views/user/settings/GeneralSettingsForm.vue'
import type {UserSettingsEdits} from '@/client/queries/account'
import {adminUserSettingsQuery, useUpdateAdminUserSettingsMutation} from '@/client/queries/admin'
import {useTitle} from '@/composables/useTitle'

const props = defineProps<{userId: number}>()

defineOptions({name: 'AdminUserSettings'})

const {t} = useI18n({useScope: 'global'})

const {data, isPending} = useQuery(computed(() => adminUserSettingsQuery(props.userId)))
useTitle(() => data.value && t('admin.users.settingsTitle', {username: data.value.user?.username}))
const updateMutation = useUpdateAdminUserSettingsMutation()

function save(edits: UserSettingsEdits) {
	return updateMutation.mutateAsync({id: props.userId, edits})
}
</script>

<style lang="scss" scoped>
.admin-user-settings__header {
	margin-block-end: 1rem;

	.title {
		margin-block-start: .5rem;
	}
}
</style>
