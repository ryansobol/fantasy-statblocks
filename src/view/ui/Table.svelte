<script lang="ts">
    import type { Monster } from "@types";
    import type { TableItem } from "src/layouts/types";
    import { stringify } from "src/util/util";

    export let monster: Monster;
    export let item: TableItem;

    function getMod(stat: number) {
        if (typeof stat != "number") return ``;
        let mod;
        if (
            item.modifier == null ||
            !item.modifier.length ||
            item.modifier == ""
        ) {
            mod = Math.floor(((stat ?? 10) - 10) / 2);
        } else {
            const func = item.modifier.contains("return")
                ? item.modifier
                : `return ${item.modifier}`;
            const customMod = new Function("stat", func);
            mod = customMod(stat);
        }
        return `(${mod >= 0 ? "+" : "-"}${Math.abs(mod)})`;
    }

    let values: any[] = monster[item.properties[0]] as any[];
    if (!Array.isArray(values)) {
        values = [];
    }

    const valueMap: Map<number, number[]> = new Map();
    for (let index = 0; index < values.length; index++) {
        const value = values[index];
        if (Array.isArray(value)) {
            for (let vIndex = 0; vIndex < value.length; vIndex++) {
                const existing = valueMap.get(vIndex) ?? [];
                existing.push(value[vIndex]);
                valueMap.set(vIndex, existing);
            }
        } else {
            valueMap.set(index, [value]);
        }
    }

    const headers = item.headers ?? [
        ...Array(values.length > 0 ? values.length : 1).keys()
    ];
</script>

<div class="table">
    {#each [...valueMap.entries()].slice(0, headers.length) as [index, values]}
        <div class="table-item">
            <span class="statblock-table-header">{headers[index]}</span>
            {#each values as value}
                <span>
                    {stringify(value)}
                    {#if item.calculate}
                        <span>
                            {getMod(value)}
                        </span>
                    {/if}
                </span>
            {/each}
        </div>
    {/each}
</div>

<style>
    .statblock-table-header {
        font-weight: var(--statblock-table-header-font-weight);
    }
    .table {
        display: flex;
        justify-content: space-evenly;
        align-items: center;
        flex-wrap: wrap;
    }
    .table-item {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-flow: column nowrap;
    }
</style>
