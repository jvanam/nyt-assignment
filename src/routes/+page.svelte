<script>
	import intro01 from '#lib/assets/intro_01_dk.jpg';
	import intro02 from '#lib/assets/intro_02_dk.jpg';
	import intro03 from '#lib/assets/intro_03_dk.jpg';
	import intro04 from '#lib/assets/intro_04_dk.jpg';
	import fig01 from '#lib/assets/fig_01_dk.jpg';
	import fig02 from '#lib/assets/fig_02_dk.jpg';
	import fig03 from '#lib/assets/fig_03_dk.jpg';
	import fig04 from '#lib/assets/fig_04_dk.jpg';
	import fig05 from '#lib/assets/fig_05_dk.jpg';

	// Collage figures: position/size in viewport units, `at` = collage progress (0–1) when it fades in.
	const figs = [
		{ src: fig01, alt: '', top: 8, left: 6, width: 26, at: 0.23 },
		{ src: fig02, alt: '', top: 14, left: 38, width: 22, at: 0.12 },
		{ src: fig03, alt: '', top: 6, left: 66, width: 28, at: 0.01 },
		{ src: fig04, alt: '', top: 52, left: 14, width: 30, at: 0.36 },
		{ src: fig05, alt: '', top: 50, left: 56, width: 26, at: 0.01 }
	];

	/** @type {HTMLElement} */ let pinnedSection;
	/** @type {HTMLElement} */ let textSection;
	/** @type {HTMLElement} */ let holdSection;
	/** @type {HTMLElement} */ let collageSection;

	let pinnedProgress = $state(0);
	let textProgress = $state(0);
	let holdProgress = $state(0);
	let collageProgress = $state(0);

	const clamp = (/** @type {number} */ n) => Math.min(1, Math.max(0, n));

	/** Progress (0–1) of scrolling through an element. */
	function progress(/** @type {HTMLElement} */ el) {
		if (!el) return 0;
		const r = el.getBoundingClientRect();
		const total = r.height - window.innerHeight;
		return Math.min(1, Math.max(0, -r.top / (total > 0 ? total : r.height)));
	}

	function update() {
		pinnedProgress = progress(pinnedSection);
		textProgress = progress(textSection);
		holdProgress = progress(holdSection);
		// Starts 30vh before the collage section enters, overlapping the end of the split fade-out.
		collageProgress = collageSection
			? clamp((window.innerHeight * .95 - collageSection.getBoundingClientRect().top) / collageSection.offsetHeight)
			: 0;
	}

	$effect(() => {
		update();
		window.addEventListener('scroll', update, { passive: true });
		window.addEventListener('resize', update);
		return () => {
			window.removeEventListener('scroll', update);
			window.removeEventListener('resize', update);
		};
	});

	// Pinned intro text: txt 1 fades over the first portion, then txt 2.
	let txt1Opacity = $derived(1 - clamp(pinnedProgress / 0.4));
	let txt2Opacity = $derived(1 - clamp((pinnedProgress - 0.5) / 0.4));

	let rightSwapped = $derived(pinnedProgress >= 0.5);
	let leftSwapped = $derived(textProgress >= 1);
	// Split layout fades out across the second half of the hold section.
	let splitOpacity = $derived(1 - Math.min(1, Math.max(0, (holdProgress - 0.5) * 2)));

</script>

<div class="collage" aria-hidden={collageProgress === 0}>
	{#each figs as fig}
		<img
			src={fig.src}
			alt={fig.alt}
			style:top="{fig.top}vh"
			style:left="{fig.left}vw"
			style:width="{fig.width}vw"
			class:hidden={collageProgress < fig.at}
		/>
	{/each}
</div>

<div class="intro_wr" style:opacity={splitOpacity} style:visibility={splitOpacity === 0 ? 'hidden' : 'visible'}>
	<div class="panel">
		<img alt="A close-up of a bouquet of flowers" src={intro01} class:hidden={leftSwapped} />
		<img alt="" src={intro04} class:hidden={!leftSwapped} />
	</div>
	<div class="panel">
		<img alt="A close-up of a bouquet of flowers" src={intro02} class:hidden={rightSwapped} />
		<img alt="" src={intro03} class:hidden={!rightSwapped} />
	</div>
</div>

<main>
	<!-- txt1 (poem) stays centered and fades, then txt2 fades, then blocks scroll. -->
	<section class="pinned_section" bind:this={pinnedSection}>
		<div class="pinned_inner">
			<div class="text_block">
				<p>
					<span class="txt1" style:opacity={txt1Opacity}>
					The trembling air, time<br/>
					sliding over our bodies like<br/>
					water. All the emptiness<br/>
					around us, as well as<br/>
					</span>
					<span class="txt2" style:opacity={txt2Opacity}>everything that remains.</span>
				</p>
			</div>
		</div>
	</section>

	<!-- Remaining text blocks scroll up normally over the left half. -->
	<section class="scroll_section" bind:this={textSection}>
		<div class="text_block">
            <p class="title_sm">OPINION</p>
            <p class="title">TO<br/>FALL<br/>IN<br/>LOVE<br/>WITH<br/>THE<br/>WORLD</p>
            <p class="poem">
                A late summer meditation<br/>
                on time, loss, and solace in<br/>
                the natural world, made from<br/>
                the work of the photographer<br/>
                Mary Manning and the<br/>
                poet Brian Turner.
            </p>
        </div>
	</section>

	<!-- Pause on intro_04, then the split layout fades out. -->
	<section class="hold_section" bind:this={holdSection}></section>

	<!-- Scroll distance over which the collage figures fade in. -->
	<section class="collage_section" bind:this={collageSection}></section>
</main>

<style>
	.intro_wr {
		width: 100%;
		height: 100dvh;
		display: grid;
		position: fixed;
		top: 0;
		left: 0;
		grid-template-columns: 50% 50%;
		transition: opacity 0.2s linear;
	}
	.panel {
		position: relative;
		height: 100dvh;
		& img {
			position: absolute;
			inset: 0;
			width: 100%;
			height: 100%;
			object-fit: cover;
			object-position: bottom;
			transition: opacity 0.8s ease;
		}
	}

	.collage {
		position: fixed;
		inset: 0;
		pointer-events: none;
		& img {
			position: absolute;
			height: auto;
			transition: opacity 0.5s ease;
		}
	}

	.hidden {
		opacity: 0;
	}

	main {
		position: relative;
	}
	.pinned_section {
		/* Scroll distance over which txt1 then txt2 fade out. */
		height: 300vh;
	}
	.pinned_inner {
		position: sticky;
		top: 0;
		height: 100vh;
		width: 50%;
		display: flex;
		align-items: center;
		padding: 0 4vw;
	}
	.pinned_inner .text_block {
		min-height: 0;
	}
	.txt1,
	.txt2 {
		transition: opacity 0.2s linear;
	}
	.scroll_section {
        mix-blend-mode: luminosity;
		width: 50%;
		padding: 0 4vw 50vh;
		/* Pull the scrolling text up so block 2 enters the viewport when
		   txt2 is at 30% opacity (pinnedProgress ≈ 0.78 over 200vh of travel). */
		margin-top: -44vh;
	}
    .title_sm{
        color: #cdcdcd;
    }
    .title{
        color: #a9a9a9;
        letter-spacing: 30%;
    }
    .poem{
        color: #EAEAEA;
    }
	.scroll_section .text_block {
		color: inherit;
	}
	.text_block {
		min-height: 100vh;
		color: var(--color-cream);
	}
	.hold_section {
		height: 200vh;
	}
	.collage_section {
		height: 400vh;
	}
     /* MIN */
	@media (max-width: 768px) {
		.intro_wr {
			grid-template-columns: 100%;
		}
		/* Only the left panel shows on mobile. */
		.panel:nth-child(2) {
			display: none;
		}
		.scroll_section {
			width: 100%;
		}
		.pinned_inner {
			width: 100%;
		}
	}
    /* MAX */
	@media (min-width: 768px) {
		p {
            font-size: 31px;
            margin-block-start: 40px;
            margin-block-end: 40px;
        }
        .title{
            font-size: 95px;
            font-family: "CheltenhamDisplay", serif;
        }
        .title_sm{
            font-size: 18px;
            font-family: "CheltenhamBold", serif;
        }
	}
</style>
