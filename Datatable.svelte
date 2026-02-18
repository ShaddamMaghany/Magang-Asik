<script lang="ts">
	import * as Pagination from '$lib/components/shadcn/ui/pagination/index.js';
	import api from '$lib/axios-instance';
	import type { DatatableColumn, PaginationData, HttpResponse, PaginationMeta } from '$lib/types';
	import type { Snippet } from 'svelte';
	import Select from './Select.svelte';
	import { ToastStore } from '$lib/stores';
	import Skeleton from './Skeleton.svelte';
	import Icon from '@iconify/svelte';
	type T = $$Generic;
	type Props<T> = {
		headerActions?: Snippet;
		columnsActions?: Snippet<[T, number]>;
		columns: DatatableColumn<T>[];
		data: T[];
		url?: string;
		responseMaping?: (data: T) => T;
		rowClick?: (data: T) => void;
		withNumbering?: boolean;
		withHeader?: boolean;
		disablePagination?: boolean;
		onFetchError?: (error: any) => void;
		searchParamKey?: string;
		class?: string;
	};
	let {
		headerActions,
		data = $bindable(),
		columns,
		url,
		columnsActions,
		responseMaping,
		rowClick,
		withNumbering = true,
		withHeader = true,
		disablePagination = false,
		onFetchError,
		searchParamKey,
		class: className
	}: Props<T> = $props();
	let pagination = $state<PaginationData>({
		page: 1,
		pageSize: 10,
		totalCount: 0,
		totalPage: 1
	});
	let loading = $state(false);
	let searchTimout = $state<any | undefined>();

	async function fetch(query?: string): Promise<T[]> {
		if (!url) return [];
		loading = true;
		try {
			const response = await api.get<HttpResponse<T[], PaginationMeta>>(url, {
				params: {
					page: pagination.page,
					size: pagination.pageSize,
					[searchParamKey || 'q']: query
				}
			});
			const meta = response.data.meta;
			pagination.page = meta?.page || 1;
			pagination.totalCount = meta?.total_count || 0;
			pagination.totalPage = meta?.total_page || 1;
			const resData = response.data.data;
			const maping = responseMaping ? resData.map(responseMaping) : resData;
			data = maping;
			return maping;
		} catch (e: any) {
			if (onFetchError) {
				onFetchError(e);
			} else {
				ToastStore.error({
					duration: 3000,
					title: 'Datatable Fetch Error',
					description: e.message || 'Failed to fetch data'
				});
			}
			return [];
		} finally {
			loading = false;
		}
	}
	function onSearch(e: Event) {
		const query = (e.target as HTMLInputElement).value;
		clearTimeout(searchTimout);
		searchTimout = setTimeout(() => {
			fetch(query);
		}, 500);
	}
	function _onClicRow(data: T) {
		rowClick && rowClick(data);
	}
	$effect(() => {
		if (!url) return;
		fetch();
	});

	export const reload = fetch;
	const pageSizeOptions = [10, 25, 50, 100];
</script>

<div class="flex flex-col gap-5 overflow-hidden {className}">
	{#if withHeader}
		<div class="flex shrink-0 items-center justify-between">
			<div
				class="flex items-center gap-2 rounded-md border border-primary-500 bg-app-bg/50 px-2 py-1"
			>
				<Select
					bind:value={pagination.pageSize}
					options={pageSizeOptions.map((size) => ({ label: size, value: size }))}
					class="h-4! w-12! py-1! text-xs"
				/>
				<span class="text-xs text-white">entries per page</span>
			</div>
			<div class="px flex w-105 items-center rounded-md border border-primary-500 bg-app-bg/50 p-2">
				<Icon icon="fluent:search-24-regular" class="mr-2 text-xl text-primary-500" />
				<input
					type="text"
					oninput={onSearch}
					placeholder="Cari"
					class="w-full bg-transparent text-sm text-white outline-none placeholder:text-[#CFDDC180]"
				/>
			</div>
			{@render headerActions?.()}
		</div>
	{/if}
	<div class="card v-scroll flex flex-1 flex-col overflow-auto px-8 py-6">
		<div class="flex-1">
			<table class="min-w-full text-center text-sm text-white">
				<thead class="text-xs text-primary-500 uppercase">
					<tr>
						{#if withNumbering}
							<th class="w-16">#</th>
						{/if}
						{#each columns as col}
							<th class="px-3 pt-3 pb-6">{col.title}</th>
						{/each}
						{#if columnsActions}
							<th class="flex justify-center px-3 pt-3 pb-6">
								<Icon icon="material-symbols:settings-outline" class="text-xl" />
							</th>
						{/if}
					</tr>
				</thead>
				<tbody>
					{#if !loading}
						{#if data.length > 0}
							{#each data as item, index}
								<tr
									onclick={(e) => {
										e.stopPropagation();
										_onClicRow(item);
									}}
									class="text-primary-50 hover:bg-white/5"
								>
									{#if withNumbering}
										<td class="text-center text-xs">
											{(pagination.page - 1) * pagination.pageSize + index + 1}
										</td>
									{/if}
									{#each columns as col}
										<td class="px-5 py-3 text-xs {col.class}">
											{#if col.render}
												{@render col.render(item)}
											{:else}
												{item[col.key]}
											{/if}
										</td>
									{/each}
									<td class="w-max px-5 py-3">
										{@render columnsActions?.(item, index)}
									</td>
								</tr>
							{/each}
						{:else}
							<tr>
								<td colspan={columns.length + 2} class="py-6 text-center text-white/50">
									No data
								</td>
							</tr>
						{/if}
					{:else}
						<tr>
							{#if withNumbering}
								<td>
									<Skeleton class="h-3 w-full" />
								</td>
							{/if}
							{#each Array.from({ length: columns.length })}
								<td class="px-5 py-4 text-xs">
									<Skeleton class="h-3 w-full" />
								</td>
							{/each}
							{#if columnsActions}
								<td>
									<Skeleton class="mx-auto h-7 w-7" />
								</td>
							{/if}
						</tr>
					{/if}
				</tbody>
			</table>
		</div>
		{#if !disablePagination}
			<div class="flex shrink-0 items-center justify-between">
				<div class="text-xs">
					Showing {data.length} of {pagination.totalCount}
					Drone devices
				</div>
				<Pagination.Root
					count={pagination.totalCount}
					perPage={pagination.pageSize}
					bind:page={pagination.page}
				>
					{#snippet children({ pages, currentPage })}
						<Pagination.Content>
							<Pagination.Item>
								<Pagination.PrevButton />
							</Pagination.Item>
							{#each pages as page (page.key)}
								{#if page.type === 'ellipsis'}
									<Pagination.Item>
										<Pagination.Ellipsis />
									</Pagination.Item>
								{:else}
									<Pagination.Item>
										<Pagination.Link {page} isActive={currentPage === page.value}>
											{page.value}
										</Pagination.Link>
									</Pagination.Item>
								{/if}
							{/each}
							<Pagination.Item>
								<Pagination.NextButton />
							</Pagination.Item>
						</Pagination.Content>
					{/snippet}
				</Pagination.Root>
			</div>
		{/if}
	</div>
</div>
