<script lang="ts">
    let { value, order, selected, special } = $props()
</script>

<div class={[`w-full border-1 rounded-md`, order[value].value  == value && "bg-green-300", selected && "border-yellow-300", special && "border-blue-200"]} style={`height:${value * 100/order.length}%`}>
</div>