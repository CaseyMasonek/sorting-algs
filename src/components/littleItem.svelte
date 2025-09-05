<script lang="ts">
    let { value, order, selected, special } = $props()
</script>

<div class={[`w-2 border-2 rounded-md`, order[value].value  == value && "bg-green-300", selected && "border-yellow-300", special && "border-red-500"]} style={`height:${2 * value}px`}>
</div>