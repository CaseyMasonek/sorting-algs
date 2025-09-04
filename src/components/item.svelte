<script lang="ts">
    let { number, order, selected } = $props()
</script>

<div class={["w-10 h-10 border-2 rounded-md text-center align-middle", order[number].value  == number && "bg-green-300", selected && "border-yellow-300"]}>
    {number}
</div>