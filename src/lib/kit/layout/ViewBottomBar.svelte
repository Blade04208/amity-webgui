<script lang="ts">
	import Button from '../Button.svelte';
	import Icon from '../Icon.svelte';
	import TextArea from '$lib/kit/Textarea.svelte';

	import {
		isCloudStorageBar,
		isContactsBar,
		isMapBar,
		isPollBar,
		isCommandBar,
		isEmojiBar,
		isStickerBar,
		isGifBar,
		setActive,
		isRecording

	} from '$lib/scripts/chatViews';
	import RecordingBar from '../RecordingBar.svelte';

	let message: string = $state('');

	const senMessage = () => {
		console.log(message);
		message = '';
	}	
</script>

<div class="viewBottomBar">
		
	<div class="recordingElements" style={
		$isRecording ? 'width: 100%; position: relative; opacity: 1; pointer-events: auto;' : 
		'width: 0px; position: absolute; opacity: 0; pointer-events: none;'
		}>
		<Button action={() => {isRecording.set(!$isRecording)}} style={3}><Icon name="X"></Icon></Button>
		<RecordingBar timestamp={$isRecording ? new Date().getTime() + 1 : new Date().getTime() + 1}></RecordingBar>
	</div>
	<Button
		width="36px"
		style={$isCloudStorageBar ? 2 : 0}
		action={() => {
			setActive("cloud")
		}}>
		<Icon name={$isCloudStorageBar ? 'X' : 'Plus'} />
	</Button>
	<TextArea 
		style="max-height: calc(100vh - 20px);"
		onkeydown={(e: KeyboardEvent) => {
			if(e.key === 'Enter' && e.shiftKey){
				console.log('shift enter');
			}else if(e.key === 'Enter'){
				e.preventDefault();
				senMessage()
			}else if(e.key === 'Escape'){
				message = '';
			}
		}}
		zIndex={12837} 
		bgc="#00000080; backdrop-filter: blur(40px);" 
		bind:value={message} 
		height={message.includes('\n') ? message.split('\n').length * 15 + 20 + 'px' : '36px'} 
		placeholder="Message General" 
		icon={message.includes('\n') ? '' : 'Chat'} 
		width="100%" 
	/>
	<div class="elements-horiz">
		<Button
			width="36px"
			style={$isContactsBar ? 2 : 0}
			action={() => {
				setActive('contacts');
		}}><Icon name={$isContactsBar ? 'X' : 'Users'} /></Button
		>
		<Button
			width="36px"
			style={$isMapBar ? 2 : 0}
			action={() => {
				setActive('maps');
			}}><Icon name={$isMapBar ? 'X' : 'Location'} /></Button
		>
		<Button
			width="36px"
			style={$isPollBar ? 2 : 0}
			action={() => {
				setActive('polls');
			}}><Icon name={$isPollBar ? 'X' : 'Equalizer'} /></Button
		>
		<Button
			width="36px"
			style={$isCommandBar ? 2 : 0}
			action={() => {
				setActive('commands');
			}}><Icon name={$isCommandBar ? 'X' : 'Terminal'} /></Button
		>
		<Button
			width="36px"
			style={$isEmojiBar ? 2 : 0}
			action={() => {
				setActive('emoji');
			}}><Icon name={$isEmojiBar ? 'X' : 'Smile'} /></Button
		>
		<Button
			width="36px"
			style={$isStickerBar ? 2 : 0}
			action={() => {
				setActive('stickers');
			}}><Icon name={$isStickerBar ? 'X' : 'StickyNotes'} /></Button
		>
		<Button
			width="36px"
			style={$isGifBar ? 2 : 0}
			action={() => {
				setActive('gifs');
			}}><Icon name={$isGifBar ? 'X' : 'GIF'} /></Button
		>
		<Button
			width="36px"
			style={$isRecording ? 1 : 0}
			action={() => {
				isRecording.set(!$isRecording)
			}}><Icon name={$isRecording ? 'Send' : 'Microphone'} /></Button
		>
	</div>
</div>

<style lang="scss">
	@use '$lib/style/colors.scss' as c;
	@use '$lib/style/variables.scss' as v;

	.recordingElements{
		transition: 0.25s;
		display: flex;
		flex-direction: row;
		gap: v.$spacing-def;
	}

	.elements-horiz{
		display: flex;
		flex-direction: row;
		gap: v.$spacing-def;
	}

	.viewBottomBar {
		height: 56px;
		bottom: 0px;
		flex-shrink: 0;
		display: flex;
		flex-direction: row;
		padding: v.$spacing-def;
		gap: v.$spacing-def;
		width: 100%;
		box-sizing: border-box;
		align-items: flex-end;
	}
</style>
