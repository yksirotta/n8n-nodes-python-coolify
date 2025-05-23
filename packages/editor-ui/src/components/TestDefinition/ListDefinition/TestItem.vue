<script setup lang="ts">
import type { TestListItem } from '@/components/TestDefinition/types';
import TimeAgo from '@/components/TimeAgo.vue';
import { useI18n } from '@/composables/useI18n';
import n8nIconButton from 'n8n-design-system/components/N8nIconButton';
import { computed } from 'vue';

export interface TestItemProps {
	test: TestListItem;
}

const props = defineProps<TestItemProps>();
const locale = useI18n();

const emit = defineEmits<{
	'run-test': [testId: string];
	'view-details': [testId: string];
	'edit-test': [testId: string];
	'delete-test': [testId: string];
	'cancel-test-run': [testId: string, testRunId: string | null];
}>();

const actions = [
	{
		icon: 'play',
		id: 'run',
		event: () => emit('run-test', props.test.id),
		tooltip: locale.baseText('testDefinition.runTest'),
		show: () => props.test.execution.status !== 'running',
	},
	{
		icon: 'stop',
		id: 'cancel',
		event: () => emit('cancel-test-run', props.test.id, props.test.execution.id),
		tooltip: locale.baseText('testDefinition.cancelTestRun'),
		show: () => props.test.execution.status === 'running',
	},
	{
		icon: 'list',
		id: 'view',
		event: () => emit('view-details', props.test.id),
		tooltip: locale.baseText('testDefinition.viewDetails'),
	},
	{
		icon: 'pen',
		id: 'edit',
		event: () => emit('edit-test', props.test.id),
		tooltip: locale.baseText('testDefinition.editTest'),
	},
	{
		icon: 'trash',
		id: 'delete',
		event: () => emit('delete-test', props.test.id),
		tooltip: locale.baseText('testDefinition.deleteTest'),
	},
];

const visibleActions = computed(() => actions.filter((action) => action.show?.() ?? true));
</script>

<template>
	<div
		:class="$style.testItem"
		:data-test-id="`test-item-${test.id}`"
		@click="$emit('view-details', test.id)"
	>
		<div :class="$style.testInfo">
			<div :class="$style.testName">
				{{ test.name }}
			</div>
			<div :class="$style.testCases">
				<n8n-text size="small">
					{{ locale.baseText('testDefinition.list.testRuns', { adjustToNumber: test.testCases }) }}
				</n8n-text>
				<template v-if="test.execution.status === 'running'">
					{{ locale.baseText('testDefinition.list.running') }}
					<n8n-spinner />
				</template>
				<span v-else-if="test.execution.lastRun">
					{{ locale.baseText('testDefinition.list.lastRun') }}
					<TimeAgo :date="test.execution.lastRun" />
				</span>
			</div>
		</div>

		<div :class="$style.metrics">
			<div :class="$style.metric">
				{{
					locale.baseText('testDefinition.list.errorRate', {
						interpolate: { errorRate: test.execution.errorRate ?? '-' },
					})
				}}
			</div>
			<div v-for="(value, key) in test.execution.metrics" :key="key" :class="$style.metric">
				{{ key }}: {{ value.toFixed(2) ?? '-' }}
			</div>
		</div>

		<div :class="$style.actions">
			<n8n-tooltip
				v-for="action in visibleActions"
				:key="action.icon"
				placement="top"
				:show-after="1000"
			>
				<template #content>
					{{ action.tooltip }}
				</template>
				<component
					:is="n8nIconButton"
					:icon="action.icon"
					:data-test-id="`${action.id}-test-button-${test.id}`"
					type="tertiary"
					size="mini"
					@click.stop="action.event"
				/>
			</n8n-tooltip>
		</div>
	</div>
</template>

<style module lang="scss">
.testItem {
	display: flex;
	align-items: center;
	padding: var(--spacing-s) var(--spacing-s) var(--spacing-s) var(--spacing-xl);
	border: 1px solid var(--color-foreground-base);
	border-radius: var(--border-radius-base);
	background-color: var(--color-background-light);
	cursor: pointer;

	&:hover {
		background-color: var(--color-background-base);
	}
}

.testInfo {
	display: flex;
	flex: 1;
	gap: var(--spacing-2xs);
}

.testName {
	display: flex;
	align-items: center;
	gap: var(--spacing-2xs);
	font-weight: var(--font-weight-bold);
	font-size: var(--font-size-s);
}

.testCases {
	color: var(--color-text-base);
	font-size: var(--font-size-2xs);
	display: flex;
	align-items: center;
	gap: var(--spacing-2xs);
}

.metrics {
	display: flex;
	gap: var(--spacing-l);
	margin: 0 var(--spacing-l);
}

.metric {
	font-size: var(--font-size-2xs);
	color: var(--color-text-dark);
	white-space: nowrap;
}

.actions {
	display: flex;
	gap: var(--spacing-xs);
	--color-button-secondary-font: var(--color-callout-info-icon);
}
</style>
