<template>
    <div>
        <p>
            弹幕播报
            <a-switch v-model="msgCheck" @change="onChange" checked-children="播报弹幕" un-checked-children="关闭播报" />
        </p>
        <p>
            舰长进场播报
            <a-switch v-model="guardCheck" @change="onChange" checked-children="欢迎舰长" un-checked-children="关闭播报" />
        </p>
        <p>
            礼物答谢播报
            <a-switch v-model="giftCheck" @change="onChange" checked-children="答谢礼物" un-checked-children="关闭播报" />
        </p>
        <p>
            醒目留言播报
            <a-switch v-model="superchatCheck" @change="onChange" checked-children="播报醒目留言" un-checked-children="关闭播报" />
        </p>
        <p><button v-on:click="testVoice">点我测试语音姬</button></p>
        <p>目前系统上可以使用的语音包有</p>
        <ul>
            <li v-for="voice in voicesTTS" :key="voice.name">
                {{ voice.name }}
            </li>
        </ul>
    </div>
</template>
<script>
import mikejson from './mike.json'
export default {
    data: function(){
        return {
            voicesTTS: null,
            msgCheck: false,
            guardCheck: true,
            superchatCheck: true,
            giftCheck: true,
            mikelist: mikejson.mikelist,
            mikeword: mikejson.mikeword
        }
    },
    danmaku: {
        'Platform.BiliBili.Service.DanmakuService.Message' (msg) {
            var text = ''
            var that = this

            var e = msg
            var d = msg.data

            function mikeExists(num){
                return (that.mikelist.indexOf(num) !== -1)
            }

            // todo: message should be cancel, with a turn off switch
            if (!d.data.isLotteryAutoMsg && d.type === 'message') {
                text = `${d.data.comment}`
                if (mikeExists(d.data.user.uid)) {return false}  
                if (that.msgCheck) { that.speak(text) }
            }
            if (d.type === 'welcomeRoomVIP') {
                text = `前方舰长${d.data.user.name}鲨过来了！`
                if (mikeExists(d.data.user.uid)) {return false}
                if (that.guardCheck) { that.speak(text) }
            }
            if (d.type === 'superchat') {
                text = `收到${d.data.user.username}的SC：${d.data.comment}`
                if (mikeExists(d.data.user.uid)) {return false}
                if (that.superchatCheck) { that.speak(text) }
            }
            if (d.type === 'gift' && d.data.gift.coinType === 'gold') {
                text = `感谢${d.data.user.username}投喂的${d.data.gift.giftName}，啾咪`
                if (mikeExists(d.data.user.uid)) {return false}
                if (that.giftCheck) { that.speak(text) }
            }
        }
    },
    computed: {
    },
    mounted () {
        var that = this

        setTimeout(()=>{that.getVoices()},3000)   
    },
    methods: {
        speak(msg, lang='zh-CN', voice) { 
            var synth = window.speechSynthesis;
            let u = new SpeechSynthesisUtterance();
            u.lang = lang;
            u.text = msg;
            u.pitch = 2;
            u.rate = 2;
            if (voice) {u.voice = voice}
            synth.speak(u);
        },
        getVoices() {
            var synth = window.speechSynthesis;
            var voices = synth.getVoices().sort(function (a, b) {
                const aname = a.name.toUpperCase(), bname = b.name.toUpperCase();
                if ( aname < bname ) return -1;
                else if ( aname == bname ) return 0;
                else return +1;
            });
            console.log('Show and list the voices on your system:', voices)
            this.voicesTTS = voices
        },
        onChange() {
            console.log(`
msg: ${this.msgCheck}
guard: ${this.guardCheck}
superchat: ${this.superchatCheck}
gift: ${this.giftCheck}
            `)
        },
        testVoice() {
            this.getVoices()
            this.speak('你好我是憨憨弹幕姬。你可以选择你系统上适用的发音包，我现在会的语言有')
            this.speak('English', 'en-US')
            this.speak('普通话','zh-CN')
            this.speak('粤语','zh-HK')
            this.speak('日本語', 'ja-JP')
            this.speak('한국의', 'ko-KR')
            // this.speak(`我不会唱歌，但我会念
            // 收到username的sc：text
            // 感谢username投喂的gifts，啾咪
            // 前方舰长username鲨过来了！
            // `)
            // this.speak('目前我的语言模板技能是恶魔Ki丽叩撒嘛教会我的，很感谢她！')
            // this.speak('Ki丽叩!Ki丽叩!发出了单推的叫声')
            this.speak('我正在学习其他的技能和多语言切换，希望我早点升级我自己')
        }
    }
}
</script>