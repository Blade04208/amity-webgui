<script lang="ts">
    import Button from '$lib/kit/Button.svelte';
	import Icon from '$lib/kit/Icon.svelte';
    import { isHttps, port, server, token } from "$lib/scripts/globalData";
	import { windowClickEvent, isFolderCtxMenu, folderClickEvent, folder } from '$lib/scripts/chatViews';
	import Textbox from '$lib/kit/Textbox.svelte';
	import Label from '$lib/kit/Label.svelte';
    import { getUser, iconList } from "$lib/scripts/requests"
    import { isFirefox } from '$lib/scripts/isFirefox';
    import ChatEntry from '../../ChatEntry.svelte';
    import { user } from '$lib/scripts/globalData';

    let{

    } = $props()

    let isChatPicker: boolean = $state(false);
    let chatPickerCtx: HTMLElement | undefined = $state();

    let icon = $state($folder.icon)
    let name = $state($folder.name)
    let elements = $state($folder.elements)
    folder.subscribe(()=>{
        icon = $folder.icon
        name = $folder.name
        elements = $folder.elements
        console.log(elements)
    })
    
    let isIconPicker = $state(false)
	let ctxMenu: HTMLElement | undefined = $state();
	let isCtxEdit: boolean = $state(false)

    let ctxDef: HTMLElement | undefined = $state();
    let ctxEdit: HTMLElement | undefined = $state();
    let iconPicker: HTMLElement | undefined = $state();

    let icons = iconList();

    isFolderCtxMenu.subscribe(()=>{
        isIconPicker = false
        isCtxEdit = false
        isChatPicker = false
    })
    let isReallyFireFox = $state<boolean>(false);
	$effect(() => {
		isReallyFireFox = isFirefox();
	});

    windowClickEvent.subscribe((e) => {
		if (
			e?.clientX < (ctxMenu?.offsetLeft as number) || 
			e?.clientX > (ctxMenu?.offsetLeft as number) + (ctxMenu?.offsetWidth as number) ||
			e?.clientY < (ctxMenu?.offsetTop as number) || 
			e?.clientY > (ctxMenu?.offsetTop as number) + (ctxMenu?.offsetHeight as number)
		) {
            isIconPicker = false
            isCtxEdit = false
			$isFolderCtxMenu = false;
		}
	});
    const folderUpdateProc = async ()=>{
        console.log(elements)
        try{
            const response = await fetch(`http${$isHttps ? "s" : ""}://${$server}:${$port}/user/me/chatfolders?token=${$token}`, {
                method: "PUT",
                body: JSON.stringify({
                    name: name,
                    icon: icon,
                    _id: $folder._id,
                    elements: elements
                }),
                headers: {
                    "Content-Type": "application/json",
                    "Access-Control-Allow-Origin": "*"
                }
            })
            console.log(response)
            getUser($isHttps, $server, $port, ($token as string))
        }catch(e){
            console.log(e)
        }
        isFolderCtxMenu.set(false)
    }

    const folderDelProc = async ()=>{
        try{
            const response = await fetch(`http${$isHttps ? "s" : ""}://${$server}:${$port}/user/me/chatfolders?token=${$token}`, {
                method: "DELETE",
                body: JSON.stringify({
                    _id: $folder._id
                }),
                headers: {
                    "Content-Type": "application/json",
                    "Access-Control-Allow-Origin": "*"
                }
            })
            console.log(response)
            getUser($isHttps, $server, $port, ($token as string))
        }catch(e){
            console.log(e)
        }
        isFolderCtxMenu.set(false)
    }

</script>

<div
    bind:this={ctxMenu}
    id="ctxMenu"
    class="window"
    style="
    position: absolute;
    left: {			
        (($folderClickEvent as MouseEvent)?.clientX < 130) ? 10 :
        isIconPicker || isChatPicker ? (($folderClickEvent as MouseEvent)?.clientX < 170) ? 10 :
        ($folderClickEvent as MouseEvent)?.clientX - 160 : 
        ($folderClickEvent as MouseEvent)?.clientX - 120
    }px;
    scale: {$isFolderCtxMenu ? 1 : 0};
    top: {$isFolderCtxMenu ? 56 + 32 + 5 : 16}px;
    z-index: 12831928471983412381931723071;
    width: {isIconPicker || isChatPicker ? 300 : 240}px;
    height: {isCtxEdit ? isIconPicker ? iconPicker?.clientHeight : isChatPicker ? chatPickerCtx?.clientHeight : ctxEdit?.clientHeight : ctxDef?.clientHeight}px;
    padding: 0;
"
>
    <div bind:this={ctxDef} class="defaultCtxMenuView" style="
        left: {isCtxEdit ? -240 : 0}px
    ">
        <div class="elements-horiz" style="justify-content: space-between; padding-right: 5px;">
            <Label icon="Folder/Default" label={name || icon}></Label>
            <Icon name={icon}></Icon>
        </div>
        <Button
            action={() => {
                isCtxEdit = true
            }}
            scaleClick={0.95}
            scaleHover={1.05}
            alignment="space-between"
            width="100%"
        >
            <div class="elem-horiz"><Icon name="Pencil/Angled"></Icon> Edit folder</div>
            <Icon name="Direction/Right"></Icon>
        </Button>
        <Button action={folderDelProc} width="100%" style={3}><Icon name="Trash"></Icon> Delete folder</Button>
    </div>
    <div bind:this={ctxEdit} class="editCtxMenuView" style="
        left: {isCtxEdit ? isIconPicker || isChatPicker ? -240 : 0 : 240}px
    ">
        <div class="menuTop">
            <Button action={()=>{
                icon = $folder.icon
                name = $folder.name
                isCtxEdit = false
            }}><Icon name="Direction/Left"></Icon></Button>
            <Textbox maxlength={32} bind:value={name} width="100%" icon="Rename" placeholder="Folder name"></Textbox>
        </div>
        <Button action={()=>{isIconPicker = true}} scaleClick={0.95} scaleHover={1.05} alignment="space-between" width="100%">
            <div class="elem-horiz"><Icon name={icon}></Icon> Icon <div style="opacity: 0.5">{icon}</div> </div>
            <Icon name="Direction/Right"></Icon>
        </Button>
        <Button action={()=>{isChatPicker = true}} scaleClick={0.95} scaleHover={1.05} alignment="space-between" width="100%">
            <div class="elem-horiz"><Icon name="Chat"></Icon> Chats <div style="opacity: 0.5">{elements.length}</div> </div>
            <Icon name="Direction/Right"></Icon>
        </Button>
        <Button action={folderUpdateProc} width="100%" style={1}><Icon name="Save"></Icon> Save</Button>
    </div>
    <div bind:this={iconPicker} class="iconPicker" style="
        left: {isIconPicker ? 0 : 240}px
    ">
        <div class="iconPickerTop">
            <Button action={()=>{isIconPicker = false}}><Icon name="Direction/Left"></Icon></Button>
            <Textbox bgc="black" width="100%" icon="Search" placeholder="Search icons..."></Textbox>
        </div>
        <grid class="iconList">
            {#each icons || [] as child}
				<Button width="36px; height: 36px;" style={(icon == child.substring(14, child.length - 4)) ? 6 : 4} action={()=>{
                    icon = child.substring(14, child.length - 4)
                    isIconPicker = false
                }}><Icon name={child.substring(14, child.length - 4)} /></Button>
			{/each}
        </grid>
    </div>
    <div bind:this={chatPickerCtx} class="chatPicker" style="
        left: {isChatPicker ? "0px" : "320px"}
    ">
        <div class="chatPickerTop">
            <Button action={()=>{isChatPicker = false}}><Icon name="Direction/Left"></Icon></Button>
            <Textbox bgc="black" width="100%" icon="Search" placeholder="Search chats..."></Textbox>
        </div>
        <div id="allChats" class="chatEntries" style="--pr: {isReallyFireFox ? 10 : 5}px">
            {#each $user?.chats || [] as child} 
                <ChatEntry click={()=>{
                    if(elements.find(e => e._id == child._id)){
                        elements = elements.filter(e => e._id != child._id)
                    }else{
                        elements = [...elements, child]
                    }
                }} isSelected={elements.find(e => e._id == child._id) ? true : false} data={child}></ChatEntry>
            {/each}
        </div>
    </div>
</div>

<style lang="scss">
    @use '$lib/style/colors.scss' as c;
	@use '$lib/style/variables.scss' as v;
    .chatPicker{
        width: 100%;
        height: 400px;
        display: flex;
        flex-direction: column;
        gap: v.$spacing-def;
        transition: 0.25s;
        position: absolute;
        top: 0;
    }

    .chatEntries{
        width: 300px;
        height: 400px;
        padding: v.$spacing-def;
        padding-top: 56px;
        box-sizing: border-box;
        display: flex;
        flex-direction: column;
        gap: v.$spacing-def;
        overflow-y: scroll;
        overflow-x: hidden;
        background-color: c.$bg;
    }

    .chatPickerTop {
        top: 0;
        left: 0;
        width: 302px;
		box-sizing: border-box;
		flex-shrink: 0;
		background: linear-gradient(to bottom, #000000ff 50%, #00000000);
		display: flex;
		flex-direction: row;
		gap: v.$spacing-def;
		position: absolute;
        padding: v.$spacing-def;
        z-index: 21374;
	}
    .iconList{
        width: 305px;
        height: 320px;
        padding: v.$spacing-def;
        padding-top: 56px;
        box-sizing: border-box;
        display: grid;
        grid-template-columns: repeat(6, 1fr);
        gap: 13px;
        overflow-y: scroll;
        overflow-x: hidden;
        background-color: c.$bg;
    }

    .iconPickerTop {
        top: 0;
        left: 0;
        width: 300px;
		box-sizing: border-box;
		flex-shrink: 0;
		background: linear-gradient(to bottom, #000000ff 50%, #00000000);
		display: flex;
		flex-direction: row;
		gap: v.$spacing-def;
		position: absolute;
        padding: v.$spacing-def;
        z-index: 21374;
	}

    .iconPicker{
        top: 0;
        width: 280px;
        display: flex;
        flex-direction: column;
        gap: v.$spacing-def;
        position: absolute;
        transition: 0.25s;
    }


    .elem-horiz {
		display: flex;
		gap: v.$spacing-def;
		align-items: center;
	}
    .menuTop{
        display: flex;
        gap: v.$spacing-def;
    }
    .editCtxMenuView{
		width: 220px;
		display: flex;
		flex-direction: column;
		gap: v.$spacing-def;
		position: absolute;
		transition: 0.25s;
        padding: v.$spacing-def;
	}
	.defaultCtxMenuView{
		width: 220px;
		display: flex;
		flex-direction: column;
		gap: v.$spacing-def;
		position: absolute;
		transition: 0.25s;
        padding: v.$spacing-def;
	}
</style>