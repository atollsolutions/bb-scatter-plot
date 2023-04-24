<script>
  import { getContext } from "svelte";
  import Scatterplot from './Scatterplot.svelte';
  export let dataProvider;
  export let dataProvider2;
  export let title;
  export let xaxis;
  export let yaxis;
  export let timeField;
  export let xaxis2;
  export let yaxis2;
  export let timeField2;
  export let xscale;
	export let yscale;
  export let ar;

  const { styleable } = getContext("sdk")
  const component = getContext("component")
</script>

<div use:styleable={$component.styles}>
    {#if !!dataProvider && !!dataProvider.rows  && !!xaxis && !!yaxis}
     <Scatterplot points={dataProvider} points2={dataProvider2} xaxis={xaxis}  yaxis={yaxis} 
       timeField={timeField} xaxis2={xaxis2}  yaxis2={yaxis2} timeField2={timeField2} xscale={xscale} yscale={yscale} ar={ar} title={title}/>
    {/if}
</div>
