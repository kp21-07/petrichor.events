<script lang="ts">
    import { getContext, onMount } from "svelte";
    import { page } from "$app/stores";
    import { invalidate, loggedIn } from "$lib/stores";
    import type { HeaderLink, Link } from '../../types';
    import Icon from 'svelte-awesome/components/Icon.svelte';
    import ellipsisV from 'svelte-awesome/icons/ellipsisV';
    import userO from 'svelte-awesome/icons/userO';
	import { fade } from "svelte/transition";
	import { goto } from "$app/navigation";

    export let title: string;
    export let links: Array<HeaderLink> = [];

    let showMenu: boolean = false;
    let popupMenuPosition: { top: number, right: number } = { top: 0, right: 10 };

    function toggleMenu() {
        showMenu = !showMenu;
        updatePopupMenuPosition();
    }

    function handleScroll() {
        if (showMenu) {
            showMenu = false;
            updatePopupMenuPosition();
        }
    }

    function handleClickOutside(event: MouseEvent) {
        const menuButton = document.querySelector('.menu-icon');
        const popupMenu = document.querySelector('.popup-menu');
        if (showMenu && !popupMenu?.contains(event.target as Node) && !menuButton?.contains(event.target as Node)) {
            showMenu = false;
            updatePopupMenuPosition();
        }
    }

    function updatePopupMenuPosition() {
        const menuButton = document.querySelector('.menu-icon');
        if (menuButton) {
            const rect = menuButton.getBoundingClientRect();
            popupMenuPosition = { top: rect.bottom + 5, right: window.innerWidth - rect.right  };
        }
    }

    let plang: string[] = [
	 'PETRICHOR',
	 'पेट्रीकोर',
	//  'பெட்ரிகோர்',
	 'પેટ્રિકોર',
	 'പെട്രിക്കോർ',
	 'পেট্রিকোর',
	 'పెట్రికోర్',
	 'ਪੈਟ੍ਰਿਕੋਰ'
	];
    let len = plang.length
	let titleDiv: HTMLElement

    onMount(() => {
        window.addEventListener('scroll', handleScroll);
        document.addEventListener('click', handleClickOutside);
        let pos = 0
		let posi = 0;
		let atLast = 0;
        setInterval(() => {
			if (pos >= plang[posi].length){
				if (atLast < 10){
					atLast++
					return
				}else {
					atLast = 0;
				let prev = posi;
				while (posi == prev){
					posi = (Math.random() * 100 ) % len | 0
				}
				pos = 0
				titleDiv.innerText = ""
			}
			}
			if (titleDiv){
				titleDiv.innerText += plang[posi].at(pos++) 
			}
		},100)
        return () => {
            window.removeEventListener('scroll', handleScroll);
            document.removeEventListener('click', handleClickOutside);
        };
    });

    $: {
        if (showMenu) {
            updatePopupMenuPosition();
        }
    }

    const displayPopUp: Function = getContext('displayPopUp')
</script>

{#if links.length > 0}
<header>
    <div class="map_info">
        ⚠️ This website is currently under development. Some features may not function as expected.
    </div>
    <div class="actual_nav" style="width: 100%; height:100%;display:flex;">
        <a class="title atmos" href="/home" bind:this={titleDiv}></a>
        <nav class="menu">
            {#each links as link,ind}
            <div>
                
                <a class="links" href={link.url} aria-label={link.linkText} on:mouseenter={
                    () => {
                        link.show = true
                    }}
                on:click = {() => {
                    if (link.linkText == "Workshop") {
                        // displayPopUp(
                            // "Message",
                            // `${link.linkText} Details will be released soon.`,
                            // 5000,
                            // ()=>{}
                            // )
                        } else if (link.url === "#") {
                            displayPopUp(
                                "Message",
                                "Coming Soon",
                                5000,
                                ()=>{}
                            )
                        }
                    }}
                on:mouseleave = {() => {
                    setTimeout(()=>{
                        link.show = false
                    },3000)
                }}
                >{link.linkText}</a>
                {#if link.show}
                <div class="child_links" transition:fade>
                    {#each link.childLinks as otherLink }
                    <a href={otherLink.url} aria-label={otherLink.linkText}>{otherLink.linkText}</a>
                    {/each}
                </div>
                {/if}
            </div>
            {/each}
        </nav>
        <a class="account" href={($invalidate || !$loggedIn) ? `/login/?to=${($page).url.pathname}` : '/profile'}>
            <Icon data={userO} scale={1.5}/>
        </a>
        <button class="menu-icon" on:click={toggleMenu} aria-label="Toggle menu">
            <Icon data={ellipsisV} scale={1.6}/>
        </button>
    </div>
</header>
    
<div class="popup-menu {showMenu ? 'show' : 'hide'}" style={`top: ${popupMenuPosition.top}px; right: ${popupMenuPosition.right}px;`}>
    <nav class="popup-nav">
        {#each links as link}
            <a href={link.url} aria-label={link.linkText} on:click={() => showMenu = false}>{link.linkText}</a>
            {#each link.childLinks as otherLink }
                <a href={otherLink.url} aria-label={otherLink.linkText}>{otherLink.linkText}</a>
                {/each}
        {/each}
    </nav>
</div>

{:else}
<header>
    <div class="title">{title}</div>
</header>
{/if}

<style>
    .actual_nav {
        padding: 0.8rem 0;
    }
    .map_info {
        width:  100%;
        background-color: rgba(3, 187, 119, 0.642);
        padding: 0.5rem 0 ;
        text-align: center;
    }
    header {
        position: fixed;
        display: flex;
        justify-content: space-around;
        align-items: center;
        flex-direction: column;
        width: 100vw;
        top: 0;
        max-height: 130px;
        z-index: 1000;
        background: rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(10px);
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .child_links{
        position: absolute;
        background-color: #fdfafa44;
        border-radius: 5px;
        top: 100%;
        transition: all 0.5s ease;
    }
    .child_links a{
        padding: 5px 0;
        font-size: 12px !important;
    }
    .child_links a:hover{
        transform: unset !important;
    }


    header::after {
        content: '';
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        height: 1px;
        margin: 0 1.5rem;
        background-color: rgba(255, 255, 255, 0.5);
    }

    .title {
        font-size: 1.2rem;
        font-weight: bolder;
        font-family: var(--sfont);
        color: #fff;
        text-wrap: nowrap;
        padding-left: 3rem;
        flex: 1;
        min-width: 200px;
    }

    .menu {
        display: flex;
        justify-content: space-evenly;
        padding: 0 5em;
        flex: 6;
    }

    .menu a {
        display: block;
        text-decoration: none;
        text-transform: uppercase;
        position: relative;
        height: 100%;
        width: 100px;
        text-align: center;
        font-size: 15px;
        color: #fff;
        font-weight: 400;
        transition: all 300ms cubic-bezier(0.075, 0.82, 0.165, 1);
        transform-origin: center;
    }

    .menu a:hover {
        transform: scale(1.2);
        text-shadow: 0 0 15px rgba(255, 255, 255, 0.8);
        font-weight: bolder;
    }

    .menu-icon {
        display: none;
        background: none;
        border: none;
        cursor: pointer;
    }

    .account {
        flex: 1;
        display: flex;
        justify-content: center;
        transition: transform 300ms cubic-bezier(0.075, 0.82, 0.165, 1), filter 300ms cubic-bezier(0.075, 0.82, 0.165, 1);
    }

    .account:hover {
        transform: scale(1.2);
        filter: drop-shadow(0 0 10px rgba(255, 255, 255, 0.8));
    }

    @media (max-width: 1000px) {
        header {
            justify-content: left;
        }

        .menu {
            display: none;
        }
        
        .menu-icon {
            display: flex;
            padding-left: 10px;
            color: #fff;
            margin-right: 10px;
            justify-content: center;
            transition: filter 300ms cubic-bezier(0.075, 0.82, 0.165, 1);
        }

        .menu-icon:hover {
            filter: drop-shadow(0 0 10px rgba(255, 255, 255, 0.8));
        }

        .title {
            flex: 8;
            padding-left: 2rem;
        }

        .account {
            flex: 1;
            padding-right: 10px;
        }

    }

    .popup-menu {
        display: flex;
        position: fixed;
        background: rgba(255, 255, 255, 0.2);
        backdrop-filter: blur(10px);
        z-index: 1001;
        border-radius: 5px;
        padding: 0 5px;
        max-height: 0;
        overflow: hidden;
        transition: max-height 0.3s ease-out, padding 0.3s ease-out;
    }

    .popup-menu.show {
        max-height: 500px;
    }

    .popup-menu.hide {
        max-height: 0;
        padding: 0;
    }

    .popup-nav {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
    }

    .popup-nav a {
        display: block;
        text-decoration: none;
        text-transform: uppercase;
        text-align: center;
        padding: 10px 0;
        font-size: 1rem;
        color: #fff;
        font-weight: bolder;
        position: relative;
        width: 100%;
    }

    .popup-nav a:not(:last-child)::after {
        content: '';
        display: block;
        width: 100%;
        height: 1px;
        background: rgba(255, 255, 255, 0.3);
        position: absolute;
        bottom: 0;
    }

    .popup-nav a:hover {
        text-shadow: 0 0 15px rgba(255, 255, 255, 0.8);
    }
</style>