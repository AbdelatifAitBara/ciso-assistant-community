<script lang="ts">
	import { complianceColorMap } from './utils';
	import { page } from '$app/stores';
	import type { z } from 'zod';
	import type { ReferenceControlSchema, ThreatSchema } from '$lib/utils/schemas';
	import { ProgressRadial } from '@skeletonlabs/skeleton';
	import { displayScoreColor, formatScoreValue } from '$lib/utils/helpers';

	export let ref_id: string;
	export let name: string;
	export let description: string;
	export let ra_id: string | undefined = undefined;
	export let threats: z.infer<typeof ThreatSchema>[] | undefined = undefined;
	export let reference_controls: z.infer<typeof ReferenceControlSchema>[] | undefined = undefined;
	export let children: Record<string, Record<string, unknown>> | undefined = undefined;
	export let canEditRequirementAssessment: boolean;
	export let status: string | undefined = undefined;
	export let statusCounts: Record<string, number> | undefined;
	export let assessable: boolean;
	export let max_score: number;

	const node = {
		ref_id,
		name,
		description,
		ra_id,
		threats,
		reference_controls,
		children,
		canEditRequirementAssessment,
		status,
		max_score,
		statusCounts,
		assessable,
		...$$restProps
	} as const;

	type TreeViewItemNode = typeof node;

	const pattern = (ref_id ? 2 : 0) + (name ? 1 : 0);
	const title: string =
		pattern == 3 ? `${ref_id} - ${name}` : pattern == 2 ? ref_id : pattern == 1 ? name : '';

	let showInfo = false;

	const getAssessableNodes = (
		startNode: TreeViewItemNode,
		assessableNodes: TreeViewItemNode[] = []
	) => {
		if (startNode.assessable) assessableNodes.push(startNode);
		if (startNode.children) {
			for (const value of Object.values(startNode.children) as TreeViewItemNode[]) {
				getAssessableNodes(value, assessableNodes);
			}
		}
		return assessableNodes;
	};

	const assessableNodes = getAssessableNodes(node);
	const hasAssessableChildren =
		children &&
		Object.keys(children).length > 0 &&
		assessableNodes.length - (node.assessable ? 1 : 0) > 0;

	const REQUIREMENT_ASSESSMENT_STATUS = [
		'compliant',
		'partially_compliant',
		'in_progress',
		'non_compliant',
		'not_applicable',
		'to_do'
	] as const;

	type StatusPercentage = {
		status: (typeof REQUIREMENT_ASSESSMENT_STATUS)[number];
		percentage: {
			value: number;
			display: string;
		};
	};

	const orderedStatusPercentages: StatusPercentage[] = REQUIREMENT_ASSESSMENT_STATUS.map(
		(status) => {
			if (!statusCounts) return { status, percentage: { value: 0, display: '0' } };
			const value = statusCounts[status] || 0;
			const percentValue: number = (value / assessableNodes.length) * 100;
			const percentage = {
				value: percentValue,
				display: percentValue.toFixed(0)
			};
			return { status, percentage };
		}
	);

	function nodeScore(): number {
		if (!statusCounts) return -1;
		let mean = statusCounts['total_score'] / statusCounts['scored'];
		return Math.floor(mean * 10) / 10;
	}

	$: classesShowInfo = (show: boolean) => (!show ? 'hidden' : '');
	$: classesShowInfoText = (show: boolean) => (show ? 'text-primary-500' : '');
	$: classesPercentText = (statusColor: string) => (statusColor === '#000000' ? 'text-white' : '');
</script>

<div class="flex flex-row justify-between space-x-8">
	<div class="flex flex-1 max-w-[80ch] flex-col">
		<span style="font-weight: 300;">
			{#if assessable && canEditRequirementAssessment}
				<span class="w-full h-full flex rounded-token hover:text-primary-500">
					<a href="/requirement-assessments/{ra_id}?next={$page.url.pathname}">
						{#if title}
							<span style="font-weight: 600;">{title}</span>
						{/if}
						{#if description}
							<p>{description}</p>
						{/if}
					</a>
				</span>
			{:else}
				<p class="max-w-[80ch] whitespace-pre-line">
					{#if title}
						<span style="font-weight: 600;">{title}</span>
						{#if assessableNodes.length > 0}
							<span class="badge variant-soft-primary">
								{assessableNodes.length}
							</span>
						{/if}
					{/if}
					{#if description}
						<p>{description}</p>
					{/if}
				</p>
			{/if}
		</span>
		{#if (threats && threats.length > 0) || (reference_controls && reference_controls.length > 0)}
			<div
				role="button"
				tabindex="0"
				class="select-none text-sm hover:text-primary-400 {classesShowInfoText(showInfo)}"
				on:click={(e) => {
					e.preventDefault();
					showInfo = !showInfo;
				}}
				on:keydown={(e) => {
					if (e.key === 'Enter') {
						e.preventDefault();
						showInfo = !showInfo;
					}
				}}
			>
				<i class="text-xs fa-solid fa-info-circle" /> Learn more
			</div>
			<div
				class="card p-2 variant-ghost-primary text-sm flex flex-row cursor-auto {classesShowInfo(
					showInfo
				)}"
			>
				<div class="flex-1">
					<p class="font-medium">
						<i class="fa-solid fa-gears" />
						Suggested reference controls
					</p>
					{#if reference_controls?.length === 0}
						<p>--</p>
					{:else if reference_controls}
						<ul class="list-disc ml-4">
							{#each reference_controls as func}
								<li>
									{#if func.id}
										<a
											class="anchor"
											href="/reference-controls/{func.id}?next={$page.url.pathname}"
										>
											{func.name}
										</a>
									{:else}
										<p>{func.name}</p>
									{/if}
								</li>
							{/each}
						</ul>
					{/if}
				</div>
				<div class="flex-1">
					<p class="font-medium">
						<i class="fa-solid fa-gears" />
						Threats covered
					</p>
					{#if threats?.length === 0}
						<p>--</p>
					{:else if threats}
						<ul class="list-disc ml-4">
							{#each threats as threat}
								<li>
									{#if threat.id}
										<a class="anchor" href="/threats/{threat.id}?next={$page.url.pathname}">
											{threat.name}
										</a>
									{:else}
										<p>{threat.name}</p>
									{/if}
								</li>
							{/each}
						</ul>
					{/if}
				</div>
			</div>
		{/if}
	</div>
	{#if hasAssessableChildren}
		<div class="flex max-w-96 grow items-center space-x-2">
			<div
				class="flex max-w-96 grow bg-gray-200 rounded-full overflow-hidden h-4 shrink self-center"
			>
				{#each orderedStatusPercentages as sp}
					<div
						class="flex flex-col justify-center overflow-hidden text-xs text-center {classesPercentText(
							complianceColorMap[sp.status]
						)}"
						style="width: {sp.percentage.value}%; background-color: {complianceColorMap[sp.status]}"
					>
						{#if sp.status !== 'to_do'}
							{sp.percentage.display}%
						{/if}
					</div>
				{/each}
			</div>
			{#if nodeScore() >= 0}
				<span>
					<ProgressRadial
						stroke={100}
						meter={displayScoreColor(nodeScore(), node.max_score)}
						font={150}
						value={formatScoreValue(nodeScore(), node.max_score)}
						width={'w-10'}>{nodeScore()}</ProgressRadial
					>
				</span>
			{/if}
		</div>
	{/if}
</div>
