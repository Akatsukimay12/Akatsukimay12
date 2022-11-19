antifake on       // to enable
antifake 1,994,48 // kick number joining with starting 1 or 994 or 48
antifake !91,!357 // only allow join number starting with 91 or 357

Example: .mention on //to enable
Example: .mention hello #mention // to update mention message
use image url for => image, sticker
use video url for => gif, mp4, sticker, audio

Example

.mention url type/sticker
.mention url type/mp4 caption
.mention url type/audio 

can add multiple urls (randomly send)

custom quoted Message Example
 Pages 10
Find a page…
Home
alive
antifake
antilink
Greetings
mention_example
plugins
prefix
sticker_pack_name
vote
Clone this wiki locally
https://github.com/lyfe00011/whatsapp-bot-md.wiki.git
Example: .mention on //to enable
Example: .mention hello #mention // to update mention message
use image url for => image, sticker
use video url for => gif, mp4, sticker, audio

Example

.mention url type/sticker
.mention url type/mp4 caption
.mention url type/audio 

can add multiple urls (randomly send)

custom quoted Message Example

in heroku avars add  MENTION: { "contextInfo": { "forwardingScore": 5, "isForwarded": true }, "linkPreview": { "head": "Test", "body": "HAHA", "mediaType": 2, "thumbnail": "https://i1.sndcdn.com/avatars-000600452151-38sfei-t500x500.jpg", "sourceUrl": "https://www.github.com/lyfe00011/whatsapp-bot-md" } , "waveform": [ 20,5,0,80,80,30,20,50 ] }


.setvar 

PREFIX:^[.,!] // for multi prefix (.,!)
PREFIX = .    //for prefix as .
PREFIX = bot  //for prefix as bot
PREFIX = null //for no prefix


.alive Hey I'm running for #uptime #quote//for text only

.alive Hi https://i.pinimg.com/736x/89/96/d2/8996d212b2cb35d5fc18898a78cad391--heart-love.jpg //for image or video

.alive ```#quote```
*uptime* : *#uptime*
#ubutton\url name# #url\button url#
#button\button# #header\header#
#cbutton\cbutton# #num\91876543210#

to see current alive message : alive 


setvar STICKER_PACKNAME: name,author // with name and author
setvar STICKER_PACKNAME: name // only name
setvar STICKER_PACKNAME: false // no pack info
OR
STICKER_PACKNAME = name,author //in heroku settings


const Heroku = require('heroku-client')
const Config = require('../config')
const { rmComma, bot, jidToNum } = require('../lib')
const heroku = new Heroku({
	token: Config.HEROKU_API_KEY,
})
const baseURI = '/apps/' + Config.HEROKU_APP_NAME

bot(
	{
		pattern: 'setsudo ?(.*)',
		fromMe: true,
		desc: 'add replied or mentioned or given num to sudo',
		type: 'heroku',
	},
	async (message, match) => {
		try {
			const vars = await heroku.get(baseURI + '/config-vars')
			const SUDO = rmComma(
				(vars.SUDO || '') +
					(!vars.SUDO ? '' : ',') +
					jidToNum(message.reply_message.jid || message.mention[0] || match)
			)
			await heroku.patch(baseURI + '/config-vars', {
				body: { SUDO },
			})
			return await message.send(
				'```' + `New SUDO Numbers are : ${SUDO}` + '```'
			)
		} catch (error) {
			console.log(error)
			return await message.send('HEROKU : ' + error.body.message, {
				quoted: message.data,
			})
		}
	}
)

bot(
	{
		pattern: 'delsudo ?(.*)',
		fromMe: true,
		desc: 'remove replied or mentioned or given num to sudo',
		type: 'heroku',
	},
	async (message, match) => {
		try {
			const vars = await heroku.get(baseURI + '/config-vars')
			const sudo = jidToNum(
				message.reply_message.jid || message.mention[0] || match
			)
			const SUDO = rmComma(vars.SUDO.replace(sudo, ''))
			await heroku.patch(baseURI + '/config-vars', {
				body: { SUDO },
			})
			await message.send('```' + `New SUDO Numbers are : ${SUDO}` + '```')
		} catch (error) {
			return await message.send(error.body.message, {
				quoted: message.data,
			})
		}
	}
)

bot(
	{
		pattern: 'getsudo ?(.*)',
		fromMe: true,
		desc: 'show sudos',
		type: 'heroku',
	},
	async (message, match) => {
		try {
			const vars = await heroku.get(baseURI + '/config-vars')
			await message.send('```' + `SUDO Numbers are : ${vars.SUDO}` + '```')
		} catch (error) {
			return await message.send('HEROKU : ' + error.body.message, {
				quoted: message.data,
			})
		}
	}
)


const { bot } = require('../lib')
bot(
	{
		pattern: 'caption ?(.*)',
		fromMe: true,
		desc: 'copy or add caption to video or image',
		type: 'whatsapp',
	},
	async (message, match) => {
		if ((message.reply_message.image || message.reply_message.video) && match)
			return await message.send(
				await message.reply_message.downloadMediaMessage(),
				{ quoted: message.quoted, caption: match },
				message.reply_message.image ? 'image' : 'video'
			)
		if (message.reply_message.text)
			await message.send(message.reply_message.text, {
				quoted: message.quoted,
			})
	}
)

const { y2mate, bot, yts, getBuffer } = require('../lib/')

bot(
	{
		pattern: 'play ?(.*)',
		fromMe: true,
		desc: 'Download youtube audio',
		type: 'download',
	},
	async (message, match) => {
		match = match || message.reply_message.text
		if (!match) return await message.send('_Example : play ghost_')
		const result = await yts(match)
		const { title } = await y2mate.get(result[0].id)
		await message.send(`_Downloading ${title}_`)
		const { buffer } = await getBuffer(await y2mate.dl(result[0].id, 'audio'))
		await message.send(buffer, { mimetype: 'audio/mpeg' }, 'audio')
	}
)


const {
	y2mate,
	bot,
	genListMessage,
	yts,
	genButtonMessage,
} = require('../lib/')

const ytIdRegex =
	/(?:http(?:s|):\/\/|)(?:(?:www\.|)youtube(?:\-nocookie|)\.com\/(?:watch\?.*(?:|\&)v=|embed|shorts\/|v\/)|youtu\.be\/)([-_0-9A-Za-z]{11})/

bot(
	{
		pattern: 'ytl ?(.*)',
		fromMe: true,
		desc: 'Download youtube video',
		type: 'download',
	},
	async (message, match) => {
		match = match || message.reply_message.text
		if (!match) return await message.send('_Example : ytl url or query_')
		if (match.startsWith('y2mate;')) {
			const [_, q, id] = match.split(';')
			const result = await y2mate.dl(id, 'video', q)
			return await message.sendFromUrl(result)
		}
		if (!ytIdRegex.test(match)) {
			const result = await yts(match)
			return await message.send(
				genListMessage(
					result.map(({ title, url, description }) => ({
						text: title,
						id: `ytl ${url}`,
						desc: description,
					})),
					'Choose Your Video',
					'DOWNLOAD'
				),
				{},
				'list'
			)
		}
		const vid = ytIdRegex.exec(match)
		const { title, video } = await y2mate.get(vid[1])
		const list = []
		for (const q in video)
			list.push({
				text: q,
				id: `ytl y2mate;${q};${vid[1]}`,
				desc: video[q].fileSizeH || video[q].size,
			})
		if (!list.length)
			return await message.send('*Not found*', {
				quoted: message.quoted,
			})
		return await message.send(
			genListMessage(list, title, 'Download'),
			{},
			'list'
		)
	}
)


const { bot, textMaker } = require('../lib')

bot(
	{
		pattern: 'sed ?(.*)',
		fromMe: true,
		desc: 'Sad text effect',
		type: 'textmaker',
	},
	async (message, match) => {
		if (!match) return await message.send('Give me text')
		const effect_url =
			'https://en.ephoto360.com/write-text-on-wet-glass-online-589.html'
		const { status, url } = await textMaker(effect_url, match)
		if (url) return await message.sendFromUrl(url)
	}
)
bot(
	{
		pattern: 'steel ?(.*)',
		fromMe: true,
		desc: 'Steel text effect',
		type: 'textmaker',
	},
	async (message, match) => {
		if (!match)
			return await message.send(
				'Give me text\nExample .steel steel;effect'
			)
		const [text1, text2] = match.split(';')
		if (!text1 || !text2)
			return await message.send(
				'Give me text\nExample .steel steel;effect'
			)
		const effect_url = 'https://en.ephoto360.com/steel-text-effect-66.html'
		const { status, url } = await textMaker(effect_url, match)
		if (url) return await message.sendFromUrl(url)
	}
)

bot(
	{
		pattern: 'metallic ?(.*)',
		fromMe: true,
		desc: 'Matallic text effect',
		type: 'textmaker',
	},
	async (message, match) => {
		if (!match) return await message.send('Give me text')
		const effect_url =
			'https://textpro.me/create-a-metallic-text-effect-free-online-1041.html'
		const { status, url } = await textMaker(effect_url, match)
		if (url) return await message.sendFromUrl(url)
	}
)
bot(
	{
		pattern: 'glitch ?(.*)',
		fromMe: true,
		desc: 'Glitch text effect',
		type: 'textmaker',
	},
	async (message, match) => {
		if (!match)
			return await message.send(
				'Give me text\nExample .glitch steel;effect'
			)
		const [text1, text2] = match.split(';')
		if (!text1 || !text2)
			return await message.send(
				'Give me text\nExample .glitch steel;effect'
			)
		const effect_url =
			'https://textpro.me/create-glitch-text-effect-style-tik-tok-983.html'
		const { status, url } = await textMaker(effect_url, match)
		if (url) return await message.sendFromUrl(url)
	}
)

bot(
	{
		pattern: 'burn ?(.*)',
		fromMe: true,
		desc: 'Matallic text effect',
		type: 'textmaker',
	},
	async (message, match) => {
		if (!match) return await message.send('Give me text')
		const effect_url =
			'https://photooxy.com/logo-and-text-effects/write-text-on-burn-paper-388.html'
		const { url } = await textMaker(effect_url, match)
		if (url) return await message.sendFromUrl(url)
	}
)

//Example for text effect with two input
bot(
	{
		pattern: '8bit ?(.*)',
		fromMe: true,
		desc: 'Glitch text effect',
		type: 'textmaker',
	},
	async (message, match) => {
		if (!match)
			return await message.send(
				'Give me text\nExample .glitch steel;effect'
			)
		const [text1, text2] = match.split(';')
		if (!text1 || !text2)
			return await message.send(
				'Give me text\nExample .glitch steel;effect'
			)
		const effect_url =
			'https://photooxy.com/logo-and-text-effects/8-bit-text-on-arcade-rift-175.html'
		const { url } = await textMaker(effect_url, match)
		if (url) return await message.sendFromUrl(url)
	}
)

const {
	forwardOrBroadCast,
	bot,
	parsedJid,
	getBuffer,
	genThumbnail,
} = require('../lib/')

const url1 = 'https://i1.sndcdn.com/avatars-000600452151-38sfei-t500x500.jpg'

bot(
	{
		pattern: 'mforward ?(.*)',
		fromMe: true,
		desc: 'forward replied msg',
		type: 'misc',
	},
	async (message, match) => {
		if (!message.reply_message)
			return await message.send('*Reply to a message*')
		if (!match)
			return await message.send(
				'*Give me a jid*\nExample .mforward jid1 jid2 jid3 jid4 ...'
			)
		const buff1 = await getBuffer(url1)
		const options = {}
		options.contextInfo = {
			forwardingScore: 5, // change it to 999 for many times forwarded
			isForwarded: true,
		}
		// ADDED /* TO REMOVE LINK PREVIEW TYPE
		options.linkPreview = {
			head: 'LyFE',
			body: '❣',
			mediaType: 2, //3 for video
			thumbnail: buff1.buffer,
			sourceUrl: 'https://www.github.com/lyfe00011/whatsapp-bot',
		}
		// ADDED */ TO REMOVE LINK PREVIEW TYPE

		options.quoted = {
			key: {
				fromMe: false,
				participant: '0@s.whatsapp.net',
				remoteJid: 'status@broadcast',
			},
			message: {
				imageMessage: {
					jpegThumbnail: await genThumbnail(buff1.buffer),
					caption: '(っ◔◡◔)っ ♥ LOVE YOU ♥',
				},
			},
		}
		if (message.reply_message.audio) {
                        options.waveform = [90,60,88,45,0,0,0,45,88,28,9]
			options.duration = 999999
			options.ptt = true // delete this if not need audio as voice always
		}
		for (const jid of parsedJid(match))
			await forwardOrBroadCast(jid, message, options)
	}
)

const { bot, getJson } = require('../lib')

bot(
	{
		pattern: 'jean ?(.*)',
		fromMe: true,
		desc: 'simple google search',
		type: 'search',
	},
	async (message, match) => {
		if (!match)
			return await message.send('*Example : jean 12 dollar in inr*')
		const { result } = await getJson(
			`https://levanter.onrender.com/jean?text=${encodeURIComponent(match)}`
		)
		if (!result) return await message.send('_Not found_')
		return await message.send('```' + result + '```')
	}
)

const { bot, getJson } = require('../lib')

bot(
	{
		pattern: 'lyrics ?(.*)',
		fromMe: true,
		desc: 'search lyrics',
		type: 'search',
	},
	async (message, match) => {
		if (!match)
			return await message.send('*Example : lyrics bhla bhla*')
		const { status ,result } = await getJson(
			`https://levanter.up.railway.app/lyrics?name=${match}`
		)
		if (!status) return await message.send('_Not found_')
		return await message.send('```' + result + '```')
	}
)


const { bot, getJson } = require('../lib')

bot(
	{
		pattern: 'time ?(.*)',
		fromMe: true,
		desc: 'find time by timeZone or name or shortcode',
		type: 'search',
	},
	async (message, match) => {
		if (!match)
			return await message.send(
				'```Give me country name or code\nEx .time US\n.time United Arab Emirates\n.time America/new_york```'
			)
		const { status, result } = await getJson(
			`https://levanter-qr.vercel.app/time?code=${encodeURIComponent(match)}`
		)
		if (!status) return await message.send(`*Not found*`)
		let msg = ''
		result.forEach(
			(zone) =>
				(msg += `Name     : ${zone.name}\nTimeZone : ${zone.timeZone}\nTime     : ${zone.time}\n\n`)
		)
		return await message.send('```' + msg.trim() + '```')
	}
)

const { bot, getJson, getBuffer } = require('../lib')

bot(
	{
		pattern: 'ig ?(.*)',
		fromMe: true,
		desc: 'Insta Profile Search',
		type: 'search',
	},
	async (message, match) => {
		if (!match)
			return await message.send('*Give me a instagram username*')
		const { result, status } = await getJson(
			`https://levanter-qr.vercel.app/ig?q=${encodeURIComponent(match)}`
		)
		if (!status) return await message.send('*not found*')
		const { name, username, avatar, posts, following, followers, description } =
			result
		const { buffer } = await getBuffer(avatar)
		await message.send(
			buffer,
			{
				caption:
					'```' +
					`username : ${username}\nname : ${name}\nbio : ${description}\nposts : ${posts}\nfollowers : ${followers}\nfollowning : ${following}` +
					'```',
			},
			'image'
		)
	}
)

const { getJson, sticker, bot } = require('../lib')

bot(
	{
		pattern: 'emix ?(.*)',
		fromMe: true,
		desc: 'mix emojis',
		type: 'search',
	},
	async (message, match) => {
		if (!match) return await message.send('Example .emix 🙂🙂')
		const { result } = await getJson(
			`https://levanter-qr.vercel.app/emix?q=${encodeURIComponent(match)}`
		)
		if (!result)
			return await message.send('_Not supported_', {
				quoted: message.data,
			})
		await message.send(
			await sticker('emix', result),
			{
				quoted: message.data,
			},
			'sticker'
		)
	}
)

const { bot, forwardOrBroadCast } = require('../lib')

bot(
	{
		pattern: 'vv ?(.*)',
		fromMe: true,
		desc: 'antiViewOnce',
		type: 'whatsapp',
	},
	async (message, match) => {
		if (!message.reply_message.image && !message.reply_message.video)
			return await message.send('*reply to a vieOnce image or video*')
		await forwardOrBroadCast(message.jid, message, { viewOnce: false })
	}
)

const { bot } = require('../lib')

bot(
	{
		pattern: 'doc ?(.*)',
		fromMe: true,
		desc: 'media to doc msg',
		type: 'whatsapp',
	},
	async (message, match) => {
		if (!message.reply_message || !message.reply_message.mimetype)
			return await message.send('Reply to a media message')
		match = !match ? 'file' : match
		match =
			match.split('.').length == 1
				? message.reply_message.audio
					? `${match}.mp3`
					: `${match}.${message.reply_message.mimetype.split('/').pop()}`
				: match
		return await message.send(
			await message.reply_message.downloadMediaMessage(),
			{ fileName: match, mimetype: message.reply_message.mimetype },
			'document'
		)
	}
)

const { bot, jidToNum } = require('../lib/')
const allowed = '919876543210' // add numbers with , not to block
bot(
	{
		on: 'text',
		fromMe: false,
		type: 'antipm',
	},
	async (message, match) => {
		if (
			!message.isGroup &&
			!message.fromMe &&
			!message.sudo &&
			!allowed.split(',').includes(jidToNum(message.participant))
		) {
			await message.send('_Personal Message Not Allowed_')
			await message.Block(message.participant)
		}
	}
)


const { bot, forwardOrBroadCast } = require('../lib')

// bot(
// 	{
// 		pattern: 'astatus ?(.*)',
// 		fromMe: true,
// 		desc: 'msg',
// 		type: 'whatsapp',
// 	},
// 	async (message, match) => {}
// )

const possibleReplies = 'send,snd,sent,snt,ayak,sd,st,ayakko,cent,cnt,cend'

bot({ on: 'text', fromMe: false, type: 'astatus' }, async (message, match) => {
	if (
		message.reply_message &&
		message.reply_message.key.remoteJid == 'status@broadcast' &&
		!message.isGroup
	) {
		for (const reply of possibleReplies.split(',')) {
			if (new RegExp(reply, 'i').test(message.text)) {
				return await forwardOrBroadCast(message.jid, message, {
					quoted: message.data,
				})
			}
		}
	}
})


const config = require('../config')
const { bot, genButtonMessage } = require('../lib/')
const Heroku = require('heroku-client')
const heroku = new Heroku({ token: config.HEROKU_API_KEY })
bot(
	{
		pattern: 'call ?(.*)',
		fromMe: true,
		desc: 'Auto reject call Manager',
		type: 'whatsapp',
	},
	async (message, match) => {
		if (!match) {
			const msg = await genButtonMessage(
				[
					{
						id: `call ${config.REJECT_CALL ? 'off' : 'on'}`,
						text: config.REJECT_CALL ? 'DISABLE' : 'ENABLE',
					},
				],
				`Auto Reject Call Manager`,
				`Auto Reject ${config.REJECT_CALL ? 'Enabled' : 'Disabled'}`
			)
			return await message.send(msg, {}, 'button')
		}
		if (match == 'on' || match == 'off') {
			try {
				await message.send(
					`_Auto Call Reject ${match == 'on' ? 'Enabled' : 'Disabled'}_`
				)
				await heroku.patch(`/apps/${config.HEROKU_APP_NAME}/config-vars`, {
					body: {
						REJECT_CALL: match == 'on' ? 'true' : 'false',
					},
				})
			} catch (error) {
				await message.send(
					`Invalid Heroku_App_Name or Heroku_Api_Key\n\n${error.message}`,
					{
						quoted: message.data,
					}
				)
			}
		}
	}


antilink on/off
antilink action/warn //delete the message and warn
antilink action/kick  // delete the message and kick
antilink action/null //only delete the message
antilink fb.com,facebook.com,twitter.com //allow only urls
antilink !fb.com,!twitter.com //not allow these urls

for only WhatsApp antilink yourGroupLink,!chat.whatsapp.com
)





