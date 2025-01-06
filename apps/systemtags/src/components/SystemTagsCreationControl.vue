<!--
  - SPDX-FileCopyrightText: 2024 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
	<div id="system-tags-creation-control">
		<h4 class="inlineblock">
			{{ t('settings', 'System tag creation') }}
		</h4>

		<p class="settings-hint">
			{{ t('settings', 'Allow only admins to create tags (enable or disable).') }}
		</p>

		<NcCheckboxRadioSwitch type="switch"
			:checked.sync="onlyAdminsCanCreateEnabled"
			@update:checked="updateSystemTagsDefault">
			{{ t('settings', 'Enable') }}
		</NcCheckboxRadioSwitch>
	</div>
</template>

<script lang="ts">
import { loadState } from '@nextcloud/initial-state'
import { showError, showSuccess } from '@nextcloud/dialogs'
import { t } from '@nextcloud/l10n'
import logger from '../logger.ts'
import { updateOnlyAdminsCanCreateSystemTags } from '../services/api.js'

import NcCheckboxRadioSwitch from '@nextcloud/vue/dist/Components/NcCheckboxRadioSwitch.js'

export default {
	name: 'SystemTagsCreationControl',

	components: {
		NcCheckboxRadioSwitch,
	},

	data() {
		return {
			onlyAdminsCanCreateEnabled: loadState('settings', 'onlyAdminsCanCreateSystemTags', '1') === '1',
		}
	},
	methods: {
		t,
		async updateSystemTagsDefault(isEnabled: boolean) {
			try {
				const responseData = await updateOnlyAdminsCanCreateSystemTags(isEnabled)
				console.debug('updateSystemTagsDefault', responseData)
				this.handleResponse({
					isEnabled,
					status: responseData.ocs?.meta?.status,
				})
			} catch (e) {
				this.handleResponse({
					errorMessage: t('settings', 'Unable to update setting'),
					error: e,
				})
			}
		},

		handleResponse({ isEnabled, status, errorMessage, error }) {
			if (status === 'ok') {
				this.onlyAdminsCanCreateEnabled = isEnabled
				showSuccess(t('settings', `System tag creation is now ${isEnabled ? 'enabled' : 'disabled'} for non-admin users`))
				return
			}

			if (errorMessage) {
				showError(errorMessage)
				logger.error(errorMessage, error)
			}
		},
	},
}
</script>
