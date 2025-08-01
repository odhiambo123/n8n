<script lang="ts">
import { ElCheckbox as Checkbox, type CheckboxValueType } from 'element-plus';
import { mapStores } from 'pinia';
import type { BaseTextKey } from '@n8n/i18n';
import { useLogStreamingStore } from '@/stores/logStreaming.store';
import { defineComponent } from 'vue';
import { useI18n } from '@n8n/i18n';

export default defineComponent({
	name: 'EventSelection',
	components: {
		Checkbox,
	},
	props: {
		destinationId: {
			type: String,
			default: 'defaultDestinationId',
		},
		readonly: Boolean,
	},
	setup() {
		const i18n = useI18n();

		return {
			i18n,
		};
	},
	data() {
		return {
			unchanged: true,
		};
	},
	computed: {
		...mapStores(useLogStreamingStore),
		anonymizeAuditMessages() {
			return this.logStreamingStore.items[this.destinationId]?.destination.anonymizeAuditMessages;
		},
	},
	methods: {
		onInput() {
			this.$emit('input');
		},
		onCheckboxChecked(eventName: string, checked: CheckboxValueType) {
			this.logStreamingStore.setSelectedInGroup(this.destinationId, eventName, Boolean(checked));
			this.$forceUpdate();
		},
		anonymizeAuditMessagesChanged(value: CheckboxValueType) {
			this.logStreamingStore.items[this.destinationId].destination.anonymizeAuditMessages =
				Boolean(value);
			this.$emit('change', { name: 'anonymizeAuditMessages', node: this.destinationId, value });
			this.$forceUpdate();
		},
		groupLabelName(t: string): string {
			return this.i18n.baseText(`settings.log-streaming.eventGroup.${t}` as BaseTextKey) ?? t;
		},
		groupLabelInfo(t: string): string | undefined {
			const labelInfo = `settings.log-streaming.eventGroup.${t}.info`;
			const infoText = this.i18n.baseText(labelInfo as BaseTextKey);
			if (infoText === labelInfo || infoText === '') return;
			return infoText;
		},
	},
});
</script>

<template>
	<div>
		<div
			v-for="group in logStreamingStore.items[destinationId]?.eventGroups"
			:key="group.name"
			shadow="never"
		>
			<!-- <template #header> -->
			<Checkbox
				:model-value="group.selected"
				:indeterminate="group.indeterminate"
				:disabled="readonly"
				@update:model-value="onInput"
				@change="onCheckboxChecked(group.name, $event)"
			>
				<strong>{{ groupLabelName(group.name) }}</strong>
				<n8n-tooltip
					v-if="groupLabelInfo(group.name)"
					placement="top"
					:popper-class="$style.tooltipPopper"
					class="ml-xs"
				>
					<n8n-icon icon="circle-help" size="small" class="ml-4xs" />
					<template #content>
						{{ groupLabelInfo(group.name) }}
					</template>
				</n8n-tooltip>
			</Checkbox>
			<Checkbox
				v-if="group.name === 'n8n.audit'"
				:model-value="logStreamingStore.items[destinationId]?.destination.anonymizeAuditMessages"
				:disabled="readonly"
				@update:model-value="onInput"
				@change="anonymizeAuditMessagesChanged"
			>
				{{ i18n.baseText('settings.log-streaming.tab.events.anonymize') }}
				<n8n-tooltip placement="top" :popper-class="$style.tooltipPopper">
					<n8n-icon icon="circle-help" size="small" class="ml-4xs" />
					<template #content>
						{{ i18n.baseText('settings.log-streaming.tab.events.anonymize.info') }}
					</template>
				</n8n-tooltip>
			</Checkbox>
			<!-- </template> -->
			<ul :class="$style.eventList">
				<li v-for="event in group.children" :key="event.name" :class="`${$style.eventListItem}`">
					<Checkbox
						:model-value="event.selected || group.selected"
						:indeterminate="event.indeterminate"
						:disabled="readonly"
						@update:model-value="onInput"
						@change="onCheckboxChecked(event.name, $event)"
					>
						{{ event.label }}
						<n8n-tooltip placement="top" :popper-class="$style.tooltipPopper">
							<template #content>
								{{ event.name }}
							</template>
						</n8n-tooltip>
					</Checkbox>
				</li>
			</ul>
		</div>
	</div>
</template>

<style lang="scss" module>
.eventListCard {
	margin-left: 1em;
}

.eventList {
	height: auto;
	overflow-y: auto;
	padding: 0;
	margin: 0;
	list-style: none;
}
.eventList .eventListItem {
	display: flex;
	align-items: center;
	justify-content: left;
	margin: 10px;
	color: var(--el-color-primary);
}

.eventListItemDisabled > {
	label > {
		span > {
			span {
				background-color: transparent !important;
				&:after {
					border-color: rgb(54, 54, 54) !important;
				}
			}
		}
	}
}

.eventList .eventListItem + .listItem {
	margin-top: 10px;
}
</style>
