<template>
	<div class="space-bottom2 clearfix">
		<div class="contain width14"><select
				class="animate button button-select pad00y truncate micro line-height15 col12 fill-lighten0"
				style="padding-right: 20px; font-style: italic;">
			<option value="">None selected</option>
			<option disabled="" value="---------">---------</option>
			<option value="Landuse">Landuse</option>
			<option value="Water">Water</option>
			<option value="Aeroways">Aeroways</option>
			<option value="Buildings">Buildings</option>
			<option value="Tunnels">Tunnels</option>
			<option value="Roads">Roads</option>
			<option value="Bridges">Bridges</option>
		</select>
			<div class="noevents width2 pin-right keyline-left ">
				<div class="pin-top icon caret-down center-y"></div>
			</div>
		</div>
		<button class="fill-purple fr width7 micro button disabled noevents">Compare</button>
	</div>
</template>