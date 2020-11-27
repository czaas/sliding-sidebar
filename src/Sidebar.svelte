<script>
	export let maxWidth = 300;
	export let position = "left";
	export let baseZIndex = 99;
	export let showVeil = true;
	export let disableDrag = false;
	export let veilStyles = `background: rgba(0,0,0,.5); {transition: opacity .2s ease;}`;
	
	let isOpen = false;
	let isDragging = false;
	let translateX = position === 'left' ? 0 : window.innerWidth;
	let movementThrow = 0;
	let veilOpacity = 0;
	
	let windowWidth = window.innerWidth;
	$: canInteract = !disableDrag && isDragging;
	
	function handleInteract() {
		isDragging = true;
	}
	function handleLetGo() {
		if (canInteract) {
			isDragging = false;
			switch(position) {
				case 'right':
					if (movementThrow > 8) {
						close();
						return;
					} else if (movementThrow < -8) {
						open();
						return;
					}
					if (translateX < windowWidth - (maxWidth / 2)){
						open();
					} else {
						close();
					}
					break;
				case 'left':
				default:
					if (movementThrow > 8) {
						open();
						return;
					} else if (movementThrow < -8) {
						close();
						return;
					}

					if (translateX > (maxWidth / 2)){
						open();
					} else {
						close();
					}
					break;
			}
		}
	}

	function handleDesktopMove(event) {
		if (canInteract) {
			const {x, movementX} = event;
			movement({ x, movementX });
		}
	}
	
	function handleMobileMove(event) {
		if (canInteract) {
			const touch = Array.from(event.touches)[0];
			if (touch) {
				movement({ x: Math.round(touch.clientX), movementX: 0 });
			}
		}
	}

	function movement({x, movementX}) {				
		movementThrow = movementX;
		switch(position) {
			case 'right':
				if (x >= windowWidth - maxWidth && x >= 0 && x <= windowWidth) {
					translateX = x;
					veilOpacity = (windowWidth - x) / maxWidth;
				} else if (x < windowWidth - maxWidth) {
					open();
				} else {
					close();
				}
				break;
			case 'left':
			default:
				if (x <= maxWidth && x >= 0) {
					translateX = x;
					veilOpacity = x / maxWidth;
				} else if (x > maxWidth) {
					open();
				} else {
					close();
				}
				break;
		}
	}

	function handleResize(e) {
		if (position === 'right') {
			translateX -= windowWidth - window.innerWidth;
			windowWidth = window.innerWidth;
		}
	}
	
	function open() {
		translateX = position === 'left' ? maxWidth : windowWidth - maxWidth;
		veilOpacity = 1;
		isOpen = true;
	}
	function close() {
		translateX = position === 'left' ? 0 : windowWidth;
		veilOpacity = 0;
		isOpen = false;
	}
	
	$: sidebarTranslateX = position === 'left' ? translateX - maxWidth : translateX;
</script>
<svelte:window 
	on:resize={handleResize}
	on:mouseup={handleLetGo}
	on:mousemove={handleDesktopMove}
	on:touchmove={handleMobileMove}
	on:touchend={handleLetGo}
/>
<div 
	class="sidebar shared" 
	class:sidebarRight={position === "right"}
	style="z-index: {baseZIndex + 1}; max-width: {maxWidth}px; transform: translate3d({sidebarTranslateX}px, 0, 0);{isDragging ? "transition: none" : ""}"
>
	<slot></slot>
</div>
<div 
	on:mousedown={handleInteract}
	on:touchdown={handleInteract}
	class="sidebarHandle shared" 
	class:sidebarRight={position === "right"}
	style="z-index: {baseZIndex + 2}; transform: translate3d({translateX}px, 0, 0);{isDragging ? "transition: none" : ""}"
>
	<slot name="handle"></slot>
</div>


{#if showVeil}
	<div
		style="opacity: {veilOpacity}; z-index: {baseZIndex}; {veilStyles}" 
		class="shared" 
		class:hideVeil={veilOpacity === 0} 
		on:click={close}
	/>
{/if}

<style>
	.shared {
		position: fixed; 
		top: 0;
		left: 0;
		width: 100%;
		height: 100vh;
	}
	.sidebar {
		max-width: 95%;
		overflow: auto;
		background: white;
		transform: translate3d(-100%, 0, 0);
		transition: transform .1s ease;
	}
	.sidebarHandle {
		width: 64px;
		background: transparent;
		position: fixed; 
		top: 0;
		left: -32px;
		transition: transform .1s ease;
		cursor: grab;
	}
	.sidebarHandle.sidebarRight {
		left: -48px;
	}
	.hideVeil {
		opacity: 0;
		pointer-events: none;
		visibility: hidden;
	}
</style>