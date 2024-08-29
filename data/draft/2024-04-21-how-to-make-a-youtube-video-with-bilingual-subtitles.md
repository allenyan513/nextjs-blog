---
title: '如何制作双语字幕的YouTube视频'
date: '2024/04/21'
lastmod: '2024/04/21'
tags: ['YouTube', '字幕', '双语字幕']
draft: false
summary: '本文介绍了如何制作双语字幕的 YouTube 视频，以及双语字幕对学习英语的重要性。'
images:
  [
    'https://lh3.googleusercontent.com/3zkP2SYe7yYoKKe47bsNe44yTgb4Ukh__rBbwXwgkjNRe4PykGG409ozBxzxkrubV7zHKjfxq6y9ShogWtMBMPyB3jiNps91LoNH8A=s500',
  ]
authors: ['default']
layout: PostLayout
---

## 1. 为什么需要双语字幕的 YouTube 视频?

[YouTube](https://www.youtube.com) 每年有数以百万计的视频被用户们上传，其中绝大多数的视频是英文的，YouTube 会自动为视频生成英文字幕,
如果有需要你还可以选择将英语翻译成其他语言的字幕，比图所示，非常方便。

![img.png](/static/images/youtub_subtitle/img_5.png)

很多研究证据表明，观看有双语字幕的 YouTube 视频， 可以极大地帮助观众学习新词汇， 因为双语字幕可以使单词的形式和含义之间的联系更加清晰，从而有利于更好地理解词汇。
此外，一份眼球追踪研究表明，观众愿意花更多时间来观看双语字幕，因为他们在阅读母语和目标语言时进行了更深入的认知处理。
这种双重接触可以通过将新词汇与熟悉的单词联系起来来强化学习，从而增强理解和记忆。
在理解方面，双语字幕可以帮助学习者更好地理解 YouTube 视频的情节，因为它们提供了可以与口语对话进行比较的直接翻译。

- [A Comparative Study of the Effect of Bilingual Subtitles and English Subtitles on College English Teaching](https://www.rcis.ro/en/section1/154-volumul-662019septembrie/2583-a-comparative-study-of-the-effect-of-bilingual-subtitles-and-english-subtitles-on-college-english-teaching.html)
- [Investigating the Effectiveness of Bilingual Subtitles for Incidental Vocabulary Learning: A Mixed Methods Study](https://discovery.ucl.ac.uk/id/eprint/10145821/)
- [Watching Subtitled Films Can Help Learning Foreign Languages](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0158409)
- [Unlocking Language Learning: The Power of Bilingual Subtitles in TV Shows and Movies](https://blogs.ntu.edu.sg/blip/unlocking-language-learning-the-power-of-bilingual-subtitles-in-tv-shows-and-movies/)

总体而言，使用双语字幕观看 YouTube 视频可以成为增强英语语言技能的有益工具，特别是对于在欣赏视频和电影的同时扩大词汇量和提高理解能力.

另外一个重要的原因是，双语字幕可以帮助视频内容的可访问性和覆盖面， 吸引更广泛的观众群,包括不同语言背景的人，
提升视频在搜索引擎中的排名和曝光度。

## 2. 如何实现在 YouTube 上观看双语字幕的视频?

我们暂时只讨论已经有字幕文件的 YouTube 视频，对于没有字幕文件的视频，我会另外写一篇来详细解释如何使用 [yt-dlp](https://github.com/yt-dlp/yt-dlp), [ffmpeg](https://ffmpeg.org/), [whisper](https://openai.com/research/whisper) 等工具来实现。

大致的实现步骤：

1. 下载 YouTube 视频的原始语言字幕文件
2. 将原始语言字幕文件翻译成目标语言字幕文件
3. YouTube 视频中添加字幕（软字幕或者硬字幕）

### 2.1 下载 YouTube 视频的原始语言字幕文件

我们将使用 [`youtube-captions-scraper`](https://www.npmjs.com/package/youtube-captions-scraper) 这个类库来下载 YouTube 视频的字幕文件， 首先我们来安装

```bash
npm i youtube-captions-scraper
```

我们只需要传入 YouTube 视频的 id 和原始语言的语言代码，就可以获取到字幕文件数据，

```javascript
import { getSubtitles } from 'youtube-captions-scraper'

const videoID = 'dJzUioqf_W0' // youtube video id
getSubtitles({ videoID: videoID, lang: 'en' }).then((captions) => {
  console.log(captions)
})
```

字幕的数据结构包含了开始时间，持续时间和字幕内容

```json
{
  "start": Number,
  "dur": Number,
  "text": String
}
```

### 2.2 将原始语言字幕文件翻译成目标语言字幕文件

有了原始语言字幕文件之后，我们就可以翻译成目标语言字幕文件了，但是我们会遇到一个技术难点:

`由于YouTube自动翻译的字幕断句并不合理，且没有标点符号，直接对每一行机器翻译的话，因为没有上下文，会导致翻译的内容差，如何优化？`

我们有这么几个解决思路：

1. 将原始语言字幕文件的文本部分合并成段，增加文本密度，然后对每一段进行机器翻译，提高翻译的质量，保证时间轴的一致性
2. 将原始语言字幕文件的文本全部拼接在一起，然后进行整体机器翻译，再通过算法将目标语言文本进行断句，保证翻译后的字幕文件和原始字幕文件的时间轴一致
3. 下载 YouTube 视频，提取音频，使用 Whisper 语音识别技术将音频转换成文本，由于 Whisper
   转写后会合理断句，并添加标点符号，然后再进行机器翻译，这样可以保证翻译的质量，且时间轴一致。
4. 人工翻译

可见目前暂时还没有特别好的解决方案，所以我们做一些取舍，权衡时间和成本，既保证翻译效果，又保证时间轴的一致性，所以目前选择方案 1

![img_1.png](/static/images/youtub_subtitle/img_1.png)

机器翻译的实现，我们可以选择如 Google Translate API、Microsoft Translator API 等。我们先来安装 [Google Translate Node.js
客户端库](https://www.npmjs.com/package/@google-cloud/translate)

```bash
npm install @google-cloud/translate
```

调用 Google Translate API 的代码如下：

```typescript
import { TranslationServiceClient } from '@google-cloud/translate'

class GoogleCloudTranslateService {
  translationServiceClient: TranslationServiceClient
  projectId: string
  location: string

  constructor() {
    this.translationServiceClient = new TranslationServiceClient()
    this.projectId = `${process.env.GOOGLE_CLOUD_PROJECT_ID}`
    this.location = 'global'
  }

  translateText = async (
    texts: string[],
    sourceLanguage: string = 'en',
    targetLanguage: string
  ) => {
    try {
      const _sourceLanguageCode = this.adapterLanguageCode(sourceLanguage)
      const _targetLanguageCode = this.adapterLanguageCode(targetLanguage)
      if (_sourceLanguageCode === _targetLanguageCode) {
        return texts
      }
      const request = {
        parent: `projects/${this.projectId}/locations/${this.location}`,
        contents: texts,
        mimeType: 'text/plain', // mime types: text/plain, text/html
        sourceLanguageCode: _sourceLanguageCode,
        targetLanguageCode: _targetLanguageCode,
      }
      const [response] = await this.translationServiceClient.translateText(request)
      if (response.translations) {
        return response.translations?.map((item) => {
          return item.translatedText || ''
        })
      }
      return null
    } catch (e) {
      console.error(e)
      return null
    }
  }

  adapterLanguageCode = (lang: string) => {
    if (lang === 'zh' || lang === 'zh-Hans' || lang === 'zh-CN') {
      return 'zh-CN'
    }
    return lang
  }
}

const googleCloudTranslateService = new GoogleCloudTranslateService()
export default googleCloudTranslateService
```

### 2.3 YouTube 视频中添加字幕（软字幕或者硬字幕）

有了原始语言字幕文件和目标语言字幕文件之后，我们就可以将它们同时输出到 YouTube 视频中了，
我们可以选择使用 Chrome 插件动态在 YouTube 视频页添加字幕(软字幕)，也可以选择使用 ffmpeg 来实现字幕与视频合并(硬字幕)。

软字幕的实现方式:
以下是 YouTube 视频页面的 DOM 结构，我们可以通过 JavaScript 来动态添加字幕元素到 `.ytp-caption-window-container` 容器中。

```html
<div class="html5-video-player" id="movie_player">
  <video class="video-stream html5-main-video"></video>
  <div class="ytp-caption-window-container" id="ytp-caption-window-container" data-layer="4"></div>
</div>
```

```javascript
var video = document.querySelector('video')
var subtitleDiv = document.getElementById('ytp-caption-window-container')
var subtitles = []

video.addEventListener('timeupdate', function () {
  const currentTime = video.currentTime
  const currentSubtitle = subtitles.find(
    (sub) => currentTime >= sub.start && currentTime <= sub.end
  )
  subtitleDiv.innerHTML = currentSubtitle ? currentSubtitle.text : ''
})
```

硬字幕的实现方式:

假设目前我们目前已经将原始语言字幕文件和目标语言字幕文件翻译好了，合并成一个.srt 文件了. 除了.srt 文件格式，还可以选择.aas
格式，这个格式支持更多的字幕样式和效果。

```text
0
00:00:00,000 --> 00:00:02,240
该视频由奥尔为您带来。
This video is brought to you by Orr.

1
00:00:02,240 --> 00:00:05,440
嗨，欢迎收看《冷聚变》的另一集。
Hi, welcome to another episode of Cold Fusion.
```

我们可以使用一下命令来将字幕文件和视频文件合并

```bash
ffmpeg -i $1 -vf subtitles=$2 -c:v libx264 -crf 23 -preset fast -c:a copy output_video.mp4
```

这里是每个选项的解释：

- -i input_video.mp4: 指定输入的视频文件。
- -vf "subtitles=subtitles_file.srt": 使用视频过滤器 -vf 来添加字幕。你需要替换 subtitles_file.srt 为你的实际字幕文件名。
- -c:v libx264: 设置视频编码器为 libx264（H.264 编码器）。
- -crf 23: 设置 CRF（恒定速率因子）为 23，这是 x264 编码的默认质量级别，质量和压缩的平衡点。
- -preset fast: 设置编码预设为 fast。预设越慢，编码速度越慢，但压缩效率越好。
- -c:a copy: 复制原始音频流，不进行音频转码。
- output_video.mp4: 指定输出文件的名字。
  确保将 $1 和 $2 替换成你的视频文件名和字幕文件名。这个命令将字幕直接烧录到视频中.

## 3. 现成的工具选择

### 3.1 Suinfy - YouTube Summarizer & Bilingual Subtitles

[Suinfy.com](https://www.suinfy.com) 是一款 Chrome 插件，免费提供了 YouTube 视频的双语字幕功能，你可以直接在 Chrome
商店中搜索 `Suinfy`
安装即可。[Chrome 商店地址](https://chromewebstore.google.com/detail/suinfy-ai-youtube-summari/kdpchhdmongbmepnnnnckkemgfndaaek)

![img_2.png](/static/images/youtub_subtitle/img_2.png)

同时 Suinfy 不单单是一款双语字幕插件，它还提供了视频摘要，视频翻译，视频字幕下载等功能，非常适合学习英语的朋友使用。
![https://lh3.googleusercontent.com/5NCvLMrDJh6POHzcWoO_WJU2nLOS64xemGtC1eYocjDqQISCNyRtGGibcxYtV06M0hRH3NwYIyZpfqJCLf7os0Cj-N8=s1280-w1280-h800](https://lh3.googleusercontent.com/5NCvLMrDJh6POHzcWoO_WJU2nLOS64xemGtC1eYocjDqQISCNyRtGGibcxYtV06M0hRH3NwYIyZpfqJCLf7os0Cj-N8=s1280-w1280-h800)

### 3.2 Memo.ac

[Memo.ac](https://memo.ac/)是一款桌面应用，提供了 YouTube 视频下载，字幕转写，字幕翻译，双语字幕和视频合并烧录等功能,
非常适合用来做双语字幕视频的制作。

![img_3.png](/static/images/youtub_subtitle/img_3.png)
