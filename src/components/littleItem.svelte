<script lang="ts">
    let { value, order, selected, special } = $props()
</script>

<div class={[`w-1 border-1 rounded-md`, order[value].value  == value && "bg-green-300", selected && "border-yellow-300", special && "border-red-500"]} style={`height:${value}px`}>
</div>