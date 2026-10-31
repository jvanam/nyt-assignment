<script>
	import intro01 from '#lib/assets/intro_01_dk.jpg';
	import intro02 from '#lib/assets/intro_02_dk.jpg';
	import intro03 from '#lib/assets/intro_03_dk.jpg';
	import intro04 from '#lib/assets/intro_04_dk.jpg';
	import fig01table from '#lib/assets/fig_01_dk.jpg';
	import fig02sofa from '#lib/assets/fig_02_dk.jpg';
	import fig03chair from '#lib/assets/fig_03_dk.jpg';
	import fig04girl from '#lib/assets/fig_04_dk.jpg';
	import fig05shirt from '#lib/assets/fig_05_dk.jpg';;

	/** @type {HTMLElement} */ let pinnedSection;
	/** @type {HTMLElement} */ let textSection;
	/** @type {HTMLElement} */ let holdSection;
	/** @type {HTMLElement} */ let collageSection;
	/** @type {HTMLElement} */ let poemEl;

	let pinnedProgress = $state(0);
	let textProgress = $state(0);
	let holdProgress = $state(0);
	let collageProgress = $state(0);
	let poemMostlyOff = $state(false);

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
		// Mobile: true once ~90% of the poem has scrolled above the top of the viewport.
		if (poemEl) {
			const pr = poemEl.getBoundingClientRect();
			poemMostlyOff = pr.top + pr.height * 0.9 < 0;
		}
	}

	$effect(() => {
		update();
		const mq = window.matchMedia('(max-width: 800px)');
		isMobile = mq.matches;
		const onMq = () => { isMobile = mq.matches; };
		mq.addEventListener('change', onMq);
		window.addEventListener('scroll', update, { passive: true });
		window.addEventListener('resize', update);
		return () => {
			mq.removeEventListener('change', onMq);
			window.removeEventListener('scroll', update);
			window.removeEventListener('resize', update);
		};
	});

	// Pinned intro text: txt 1 fades over the first portion, then txt 2.
	let txt1Opacity = $derived(1 - clamp(pinnedProgress / 0.4));
	let txt2Opacity = $derived(1 - clamp((pinnedProgress - 0.5) / 0.4));

	let rightSwapped = $derived(pinnedProgress >= 0.5);
	// Desktop swaps when the scroll section is fully scrolled; mobile holds the
	// first image until most of the poem has left the screen.
	let isMobile = $state(false);
	let leftSwapped = $derived(isMobile ? poemMostlyOff : textProgress >= 1);
	// Split layout fades out across the second half of the hold section.
	let splitOpacity = $derived(1 - Math.min(1, Math.max(0, (holdProgress - 0.5) * 2)));

</script>

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

	<section class="scroll_section" bind:this={textSection}>
		<div class="text_block">
            <p class="title_sm">OPINION</p>
            <p class="title">TO<br/>FALL<br/>IN LOVE<br/>WITH<br/>THE<br/>WORLD</p>
            <p class="poem" bind:this={poemEl}>
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

<div class="collage_mb" aria-hidden={collageProgress === 0}>
	<img src={fig03chair} class="fig3" alt='' class:hidden={collageProgress < 0.001}>
	<img src={fig04girl} class="fig4" alt='' class:hidden={collageProgress < 0.80}/>
	<div class="fig_spacer"></div>
	<img src={fig01table} class="fig1" alt='' class:hidden={collageProgress < 0.5}/>
	<img src={fig02sofa} class="fig2" alt='' class:hidden={collageProgress < 0.25}/>
</div>

<div class="collage_dk" aria-hidden={collageProgress === 0}>
	<img src={fig03chair} class="fig3" alt='' class:hidden={collageProgress < 0.01}>
	<img src={fig04girl} class="fig4" alt='' class:hidden={collageProgress < 0.80}/>
	<img src={fig02sofa} class="fig2" alt='' class:hidden={collageProgress < 0.3}/>
	<div class="fig_spacer"></div>
	<img src={fig01table} class="fig1" alt='' class:hidden={collageProgress < 0.3}/>
	<img src={fig05shirt} class="fig5" alt='' class:hidden={collageProgress < 0.01}/>
</div>

<style>
	.intro_wr {
		width: 100%;
		height: 100vh;
		display: grid;
		position: fixed;
		top: 0;
		left: 0;
		grid-template-columns: 50% 50%;
		transition: opacity 0.2s linear;
	}
	.panel {
		position: relative;
		height: 100vh;
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
		font-family: "CheltenhamDisplay", serif;
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
     /* MIN */
	@media (max-width: 800px) {
		p {
            font-size: 23px;
            margin-block-start: 30px;
            margin-block-end: 30px;
        }
		.title {
			color: #ececec;
            font-size: 65px;
        }
        .title_sm {
            font-size: 16px;
            font-family: "CheltenhamBold", serif;
        }
		.intro_wr {
			grid-template-columns: 100%;
		}
		.panel:nth-child(2) {
			display: none;
		}
		.scroll_section {
			width: 100%;
			display: grid;
			justify-content: center;
			/* margin-left: 50%; */
			/* transform: translateX(-50%); */
		}
		.pinned_section {
			display: grid;
			justify-content: center;
		}
		.pinned_inner {
			width: 100%;
		}
		.collage_section {
			height: 300vh;
		}
		.collage_dk{
			display:none;
		}
		.collage_mb {
			width: min(100%, calc(100vh * 2 / 3));
			height: auto;
			aspect-ratio: 2/3;
			top: 50%;
			left: 50%;
			transform: translate(-50%, -50%);
			display: flex;
			align-items: flex-start;
			align-content: flex-start;
			justify-content: flex-start;
			flex-wrap: wrap;
			position: fixed;
			pointer-events: none;
			& img {
				transition: opacity 0.5s ease;
				display: block;
			}
		}
		.fig1, .fig2, .fig3, .fig5 {
			width: calc(200% / 7);
			height: auto;
		}
		.fig4{
			width: calc(500% / 7);
		}
		.fig_spacer {
			height: 10px;
			width: calc(200% / 7);
		}
	}
    /* MAX */
	@media (min-width: 800px) {
		p {
            font-size: 31px;
            margin-block-start: 40px;
            margin-block-end: 40px;
        }
        .title {
        	color: #a9a9a9;
            font-size: 95px;
        }
        .title_sm {
            font-size: 18px;
            font-family: "CheltenhamBold", serif;
        }
		.scroll_section {
			width: 100%;
			padding: 0 4vw 50vh;
		}
		.collage_section {
			height: 300vh;
		}
		.collage_mb {
			display: none;
		}
		.collage_dk {
			height: 100vh;
			width: calc((100vh * 2 / 3) + (100vh / 6) + 10px);
			left: 50%;
			display: flex;
			flex-wrap: wrap;
			transform: translateX(-50%);
			top: 0px;
			position: fixed;
			/* inset: 0; */
			pointer-events: none;
			& img {
				transition: opacity 0.5s ease;
				display: block;
			}
		}
		.fig1, .fig2, .fig3, .fig5 {
			height: 25vh
		}
		.fig4{
			height: 75vh;
		}
		.fig_spacer {
			height: 25vh;
			width: calc(25vh * 2 / 3);
		}
	}
</style>
