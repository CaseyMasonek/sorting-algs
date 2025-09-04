<script lang="ts">
    let { value, order, selected, special } = $props()
</script>

<div class={["w-10 h-10 border-2 rounded-md text-center align-middle", order[value].value  == value && "bg-green-300", selected && "border-yellow-300", special && "border-red-500"]}>
    {value}
</div>