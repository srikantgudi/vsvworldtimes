<script>
	import {onMount} from 'svelte';
	import moment from 'moment-timezone';
	let name = 'world';
	let continents = [];
	let continent = '';
	let zones = [];
	let zone = '';
	let ztime;
	let ang = {h:0, m:0,s:0};
	let comarks = [];
	let zmarks = [];
	let colen = 42;
	let zonelen = 53;
	let handcolor = 'navy';
	let zsearch = '';
	const textpoints = (src, len) => {
		let arr = [];
		let ncount = src.length;
		let ang = 360 / ncount;
		for (let c=0; c < ncount;c++) {
			let rad = (c * ang - 90) * Math.PI / 180;
			arr = [
				...arr,
				{
					x: Math.cos(rad) * len,
					y: Math.sin(rad) * len,
					val: src[c]
				}
			]
		}
		return arr;
	}
	onMount(() => {
		let cos = new Set();
		moment.tz.names().map(z => {
			const co = z.split('/')[0];
			if (co && co.match(/[AE][a-z]{3,}/)) {
				cos.add(co);
			}
		});
		continents = [...cos];
		console.log('znames:', continents);
		continent = continents[0];
		const coang = 360 / continents.length;
		comarks = textpoints(continents, colen);
		console.log('comarks:', comarks);
		getZones(continent);
		zmarks = textpoints(zones, zonelen)
		setZtime(zones[0]);
	});
	
	$: getZones = (co) => {
		continent = co;
		zones = moment.tz.names().filter(f => {
			const arr = f.split('/');
			return co === arr[0]
		}).map(z => z.split('/')[1]).filter(f => f.match(/[a-z]{4,}/))
		zmarks = textpoints(zones,zonelen);
		setZtime(zones[0]);
	}
	const setZtime = (z) => {
		zone = `${continent}/${z}`;
		ztime = moment().tz(zone);
		const hr = ztime.hour();
		const mi = ztime.minute();
		const se = ztime.second();
		ang = {
			h: (hr * 30 + mi / 2) - 90,
			m: (mi * 6 + se / 10) - 90,
			s: (se * 6) - 90
		}
	}
</script>

<style>
	* {
		font-family: Helvetica;
		font-weight: 400;
	}
	.link {
		cursor: pointer;
	}
	.app-title {
		font-size:28px;
	}
	.subtitle {
		font-size: 12px;
	}
	main {
		padding: 0.25em 2em;
		display: flex;
		align-items: center;
		flex-direction: column;
		justify-content: space-around;
	}
	.hand {
		transition: all 0.5s;
	}
	.subtitle:hover {
		transform: scale(1.5);
		transition: all 0.5s;
	}
	svg:hover {
		transform: scale(1.5);
	}
</style>

<main>
	<div style="text-align:center;">
		<div class="app-title">World Times</div>
		<div class="subtitle">
			Click on continents (inner circle) to view zones of the selected continent
		</div>
		<div class="subtitle">
			Hover on zones (out circle) to view zone time of the selected zone
		</div>
	</div>
	<div style="margin: 5em">
		<div>
			<svg width={600} viewBox="-60 -60 120 120">
				<circle r={50} cx={0} cy={0} fill="#eef" />
				{#each comarks as cm}
				<text class="link" on:click={() => getZones(cm.val)} x={cm.x} y={cm.y} text-anchor="middle" font-size={continent===cm.val ? 5 : 3} font-weight={continent===cm.val ? '600' : '400'} fill={continent===cm.val ? 'blue': 'grey'}>{cm.val}</text>
				{/each}
				{#each zmarks as zm}
				<text class="link" on:mouseover={() => setZtime(zm.val)} x={zm.x} y={zm.y} text-anchor="middle" font-size={zone===zm.val ? 2.5 : 1.5}>{zm.val.substr(zm.val.indexOf('/')+1)}</text>
				{/each}
				<text x={0} y={-20} font-size={5} text-anchor="middle">{zone}</text>
				<text x={0} y={10} font-size={4} text-anchor="middle">{ztime?.format('ddd DD MMM HH:mm:ss')}</text>
				<line class="hand" x1={-6} x2={30} stroke={handcolor} transform={`rotate(${ang.h} 0 0)`} />
				<line class="hand" x1={-6} x2={45} stroke={handcolor} transform={`rotate(${ang.m} 0 0)`} />
				<line class="hand" x1={-6} x2={48} stroke="#999" stroke-width="0.5" transform={`rotate(${ang.s} 0 0)`} />
			</svg>
		</div>
	</div>
</main>
