<!--
@author: wuyuxin
@description: 选择汉字验证码
@time: 2021/1/19
-->
<template>
    <div
            class="zkr"
            style="position: relative"
            @mouseover="mode === 'pop' ? (showImage = true) : undefined"
            @mouseout="mode === 'pop' ? (showImage = false) : undefined"
    >
        <div class="verify-bar-area"
             :style="{width: barSize.width,height: barSize.height,'border-color': barAreaBorderColor,'line-height': barSize.height}">
            <i class="layui-icon layui-icon-vercode" style="font-size: 18px; color: #9198a3;"></i>
            <span class="verify-msg"
            ><span v-if="spanshow" class="size">
                请依次点击
            </span>
        <p :class="{fontColor: barAreaColor === 1,fontColor1: barAreaColor === 2,fontColor2: barAreaColor === 3,}">
          {{ text }}
        </p>
      </span>
            <span class="refresh" @click="refresh">刷新</span>
        </div>
        <div
                class="verify-img-out"
                v-show="showImage"
                @mouseover="mode === 'pop' ? (showImage = true) : undefined"
                @mouseout="mode === 'pop' ? (showImage = false) : undefined"
                :style="
        mode === 'pop'
          ? {
              display: 'none',
              position: 'absolute',
              bottom: '42px',
              height: parseInt(imgHeight) + vSpace + 'px',
            }
          : {
              position: 'relative',
              height: parseInt(imgHeight) + vSpace + 'px',
            }
      "
        >
            <div
                    class="verify-img-panel"
                    :style="{
          width:imgWidth,
          height:imgHeight,
          'background-size': imgWidth + ' ' + imgHeight,
          'margin-bottom': vSpace + 'px',
        }"
            >
                <!-- <div class="verify-refresh" style="z-index:3" @click="refresh" v-show="showRefresh">
                            <i class="iconfont icon-refresh"></i>
                        </div> -->
                <canvas
                        :width="imgWidth"
                        :height="imgHeight"
                        ref="canvas"
                        @click="bindingClick ? canvasClick($event) : undefined"
                ></canvas>
                <div
                        v-for="(tempPoint, index) in tempPoints"
                        :key="index"
                        class="point-area"
                        :style="{
            'background-color': '#1abd6c',
            color: '#fff',
            'z-index': 9999,
            width: '30px',
            height: '30px',
            'text-align': 'center',
            'line-height': '30px',
            'border-radius': '50%',
            position: 'absolute',
            top: parseInt(tempPoint.y - 10) + 'px',
            left: parseInt(tempPoint.x - 10) + 'px',
          }"
                >
                    {{ index + 1 }}
                </div>
            </div>
        </div>
    </div>
</template>
<script type="text/babel">
    /**
     * VerifyPoints
     * @description 点选
     * */

    import {resetSize} from "./util.js";

    function getOffset(Node, offset = {top: 0, left: 0}) {
        if (Node == document.body) {
            //当该节点为body节点时，结束递归
            return offset;
        }
        offset.top += Node.offsetTop;
        offset.left += Node.offsetLeft;
        return getOffset(Node.parentNode, offset); //向上累加offset里的值
    }

    export default {
        name: "VerifyPoints",
        props: {
            //弹出式pop，固定fixed
            mode: {
                type: String,
                default: "fixed",
            },
            //默认的文字数量
            defaultNum: {
                type: Number,
                default: 4,
            },
            //校对的文字数量
            checkNum: {
                type: Number,
                default: 3,
            },
            //间隔
            vSpace: {
                type: Number,
                default: 5,
            },
            //图片地址
            imgUrl: {
                type: String,
                default: 'https://picsum.photos/300/150/?image='
            },
            //图片数组
            imgName: {
                type: Array,
                default() {
                    return ["350x150", "350x450"];
                },
            },

            //图片大小
            imgSize: {
                type: Object,
                default() {
                    return {
                        width: "400px",
                        height: "200px",
                    };
                },
            },
            //sanvas 宽度
            sanvasWidth: {
                type: Number,
                default: 300
            },
            //sanvas 高度
            sanvasHeight: {
                type: Number,
                default: 150
            },
            //滑动条大小下
            barSize: {
                type: Object,
                default() {
                    return {
                        width: "300px",
                        height: "40px",
                    };
                },
            },
        },
        data() {
            return {
                spanshow: true,
                fontPos: [], // 选中的坐标信息
                checkPosArr: [], //用户点击的坐标
                num: 1, //点击的记数
                imgRand: 0, // //随机的背景图片
                imgHeight: this.sanvasHeight,
                imgWidth: this.sanvasWidth,
                barHeight: 0,
                barWidth: 0,
                showImage: true,
                tempPoints: [],
                text: "",
                barAreaColor: 1,
                barAreaBorderColor: undefined,
                showRefresh: true,
                bindingClick: true,
            };
        },
        computed: {
            resetSize() {
                return resetSize;
            },
        },
        methods: {
            //生成随机整数
            rInt(max) {
                return Math.floor((Math.random() * 100000) % max);
            },
            getRandomNumberByRange(start, end) {
                return Math.round(Math.random() * (end - start) + start)
            },
            //生成随机颜色
            rColor() {
                let a = ((Math.random() * 5 + 5) / 10).toFixed(2);
                return `rgba(${this.rInt(256)}, ${this.rInt(256)}, ${this.rInt(
                    256
                )}, ${a})`;
            },
            init() {
                //加载页面
                this.fontPos.splice(0, this.fontPos.length);
                this.checkPosArr.splice(0, this.checkPosArr.length);
                this.num = 1;
                this.imgRand = Math.floor(Math.random() * this.imgName.length);
                this.$nextTick(() => {
                    var obj = this.resetSize(this); //重新设置宽度高度
                    this.refresh();
                    this.$parent.$emit("ready", this);
                });
            },
            canvasClick(e) {
                this.checkPosArr.push(this.getMousePos(this.$refs.canvas, e));
                console.log(2222,this.checkPosArr);
                console.log(33333,this.num);
                console.log(this.checkNum)
                if (this.num == this.checkNum) {
                    this.num = this.createPoint(this.getMousePos(this.$refs.canvas, e));
                    setTimeout(() => {
                        console.log('fontPos',this.fontPos);
                        console.log('checkPosArr',this.checkPosArr);
                        var flag = this.comparePos(this.fontPos, this.checkPosArr);
                        if (flag == false) {
                            //验证失败
                            this.barAreaColor = 2;
                            this.barAreaBorderColor = "#d9534f";
                            this.spanshow = false;
                            this.text = "验证失败";
                            this.$emit('codePrice','验证失败')
                            setTimeout(() => {
                                this.refresh();
                            }, 500);
                        } else {
                            //验证成功
                            this.barAreaColor = 3;
                            this.barAreaBorderColor = "#5cb85c";
                            this.spanshow = false;
                            this.text = "验证成功";
                            this.$emit('codePrice','验证成功')
                            this.showRefresh = false;
                            this.bindingClick = false;
                        }
                    }, 400);
                }

                if (this.num < this.checkNum) {
                    this.num = this.createPoint(this.getMousePos(this.$refs.canvas, e));
                }
            },
            getRandomImgSrc() {
                return this.imgUrl + this.getRandomNumberByRange(0, 1084)

                // return 'https://picsum.photos/id/407/300/150'
            },
            //绘制合成的图片
            drawImg: function (obj, img) {
                //准备canvas环境
                var canvas = this.$refs.canvas;
                var ctx = canvas.getContext("2d");
                // 绘制图片
                ctx.drawImage(
                    img,
                    0,
                    0,
                    parseInt(this.imgWidth),
                    parseInt(this.imgHeight)
                );

                // 绘制水印
                var fontSizeArr = [
                    "italic small-caps bold 20px microsoft yahei",
                    "small-caps normal 25px arial",
                    "34px microsoft yahei",
                ];
                var fontStr =
                    "天地玄黄宇宙洪荒日月盈昃辰宿列张寒来暑往秋收冬藏闰余成岁律吕调阳云腾致雨露结为霜金生丽水玉出昆冈剑号巨阙珠称夜光果珍李柰菜重芥姜海咸河淡鳞潜羽翔龙师火帝鸟官人皇始制文字乃服衣裳推位让国有虞陶唐吊民伐罪周发殷汤坐朝问道垂拱平章爱育黎首臣伏戎羌遐迩体率宾归王乙 一 匕 卜 刀 刁 丁 二 了 力 入 人 又 干 才 叉 土 川 寸 大 勺 凡 廾 孓 己 久 弓 口 工 及 巾 孑 女 乞 廿 尢 千 三 山 丫 亡 丸 尸 刃 士 夕 已 上 巳 小 也 于 下 兀 之 子 弋 丈 巴 分 比 卞 不 尺 仇 丑 歹 丹 弔 吊 仃 斗 厄 反 介 亢 氏 方 勾 勻 丐 互 戈 化 乏 今 幻 公 父 夫 井 木 欠 毛 戶 冗 孔 日 內 仁 卅 匹 斤 手 切 火 什 支 牛 毋 升 片 勿 午 壬 仍 夭 月 兮 少 太 水 王 犬 屯 爪 心 尹 允 四 牙 天 刈 文 予 引 止 爻 尤 友 仄 元 中 曰 凹 叭 白 半 包 北 本 平 必 弁 丙 布 冊 斥 叱 出 司 匆 且 代 旦 石 冉 叨 氐 叼 叮 冬 禾 伋 功 弗 付 弘 可 瓜 叻 古 句 叫 市 甘 兄 卡 加 刊 甲 立 穴 另 令 目 末 乎 巧 卉 巨 囚 申 丘 矛 史 叩 奴 皿 仞 民 叵 奶 占 尼 疋 仟 皮 失 丕 示 去 仙 他 外 五 卯 戊 央 矢 母 玄 田 幼 未 由 甩 它 仕 以 印 召 生 世 矽 札 主 玉 左 永 瓦 乍 孕 仗 用 只 朮 仔 正 匝 右 安 仰 扒 百 仳 份 并 冰 多 宅 臣 丞 吃 弛 她 充 圳 舛 次 此 存 吋 打 地 至 汀 丟 帆 而 氾 行 耒 伐 伕 仿 耳 奸 舌 合 好 圭 圾 匠 阬 尖 妃 朵 灰 艮 囝 共 亙 件 各 犯 尬 缶 屹 交 伉 回 亥 光 列 羽 伏 匡 岌 如 劣 牟 乒 老 臼 汍 米 任 乓 吉 乩 全 旬 吏 牝 企 糸 色 六 氖 式 考 似 寺 危 戎 汁 刎 年 西 血 曲 戍 向 肉 收 有 戌 匈 羊 刑 凶 旭 朱 兇 吐 吸 朽 同 凸 妁 圯 吁 汙 名 字 死 休 守 夙 迆 扔 圩 妄 舟 先 因 吆 吒 州 伍 自 宇 聿 夷 在 再 亦 仲 竹 曳 伊 早 衣 旨 兆 呆 呃 吧 伯 伴 孛 貝 庇 妣 佛 釆 杓 別 兵 步 佈 材 岑 岔 佔 吵 池 車 辰 成 呈 盯 赤 呎 沖 助 串 床 囪 吹 伺 村 忖 但 低 弟 佃 甸 佚 甬 豆 杜 妒 兌 妨 坊 估 谷 告 究 囤 改 即 邑 佝 芣 彷 吩 否 汎 汗 更 決 汞 灸 均 吟 見 冷 孚 忌 杠 杆 位 判 坎 甫 吠 囫 肓 扛 坩 牢 吼 含 妞 君 宏 角 坑 戒 何 卵 忙 罕 里 伽 夾 利 困 沍 局 求 江 攻 汲 扣 李 旱 劫 尿 弧 亨 妓 妊 克 伶 吶 完 足 七 身 劬 吝 沒 牡 妖 弄 免 杗 妙 男 良 羌 忍 巡 你 豕 汛 屁 宋 杞 杉 序 佞 每 吾 托 呂 束 姒 辛 刪 吞 汕 岐 妥 圻 攸 抆 彤 巫 牠 私 坍 匣 汝 杏 吱 佗 呀 吻 孝 圬 忘 努 希 污 矣 尾 妝 汐 妍 役 伸 兕 酉 余 我 佑 吳 灼 壯 走 形 卮 作 佇 皂 住 坐 杖 禿 吮 言 佐 冶 址 孜 秀 災 妤 姊 呵 哎 艾 奇 岸 昂 把 八 爸 杷 佰 版 板 扳 扮 並 姅 杯 卑 彼 枇 沘 汴 表 秉 併 枋 帛 附 兔 昌 尚 長 抄 炒 扯 坼 沉 忱 枕 沈 承 侈 泜 祇 妯 扭 初 杵 垂 炊 沌 刺 卒 昕 隹 昔 岱 到 受 的 底 狄 店 典 佻 定 咚 東 抖 投 枓 竺 泛 亞 扼 岡 抗 居 疙 咎 汽 房 玖 汾 非 垃 姑 放 忿 孤 炕 姐 呱 奉 咄 玨 糾 果 剁 乖 肯 枉 府 官 昆 具 劾 氛 臾 扶 沆 咖 汪 其 杲 青 亟 兒 咕 儿 呷 狂 快 和 固 穹 技 刻 虎 斧 阜 卦 坷 例 忽 侖 來 肌 昏 軋 刮 彿 疚 屆 赳 佬 庚 況 供 季 咐 沬 杭 或 肋 抉 歿 妳 金 京 侃 屈 昊 苧 卷 帕 冽 找 弩 咀 妾 妮 拋 沛 肩 戕 杼 沁 明 戾 念 券 咆 呼 佳 奈 孟 呶 帑 庖 空 爬 門 幸 汶 劻 忸 批 坤 剌 取 杪 命 歧 味 呢 疝 姍 兩 甽 枚 抒 妻 虱 些 氓 呻 於 岷 枝 社 妹 佺 事 始 羋 佩 所 沫 欣 坦 沙 宜 盲 卹 汰 坡 林 旺 叔 刷 昇 折 汨 呸 牧 姆 坪 忝 委 侍 炎 舍 朋 沐 沃 奄 使 秈 協 物 臥 乳 析 宛 盂 依 祀 帖 享 沂 易 姓 岳 佾 玥 咒 佯 弦 杳 刖 沅 直 昀 狀 炙 制 抓 夜 咋 罔 岫 往 卓 武 侑 怎 侏 爭 忠 忪 知 肴 抑 周 雨 宙 宗 哀 拗 拔 疤 拜 拌 抱 保 炮 背 怫 波 盃 甭 泵 抨 泌 拂 便 扁 砭 封 拚 屏 柄 炳 泊 勃 怕 拍 哺 怖 拆 柵 查 差 剎 柴 沾 疢 拖 柢 沱 治 重 盅 抽 怵 穿 春 促 斫 奏 怛 待 怠 殆 毒 抵 帝 牴 約 芍 拈 酊 訂 峒 度 肚 段 耑 盾 盹 法 飛 盼 風 韭 契 缸 建 哇 垓 赴 狗 孩 疥 俊 級 拘 勁 姜 俄 曷 迴 虹 柙 肝 姣 負 皈 狙 侯 虺 奐 枸 咳 咯 九 恆 前 故 宦 革 剋 沸 癸 氟 祈 奎 沮 柯 韋 侷 娃 垠 疳 巷 狐 既 徊 哈 垢 矜 拎 紅 看 洸 紀 牯 姦 哄 很 枴 紇 芒 架 拒 陘 拐 俐 柑 押 冠 砍 沽 肛 訃 皇 界 柳 泓 俘 拉 虻 昤 卻 冒 皆 俚 咧 眇 後 姘 柬 厚 俏 觔 虐 怒 姥 枯 竿 哪 趴 抹 河 怩 柒 為 盆 軌 泣 計 科 荊 亮 后 恰 玟 昧 祕 屎 急 叛 耐 秒 炯 紆 品 咬 舢 泡 秋 勉 招 泖 某 匍 客 衹 泠 玫 泥 是 咪 弭 俞 酋 抿 侶 披 染 炬 迺 削 美 軍 紉 咩 衫 泄 首 哂 毗 甚 眉 咽 食 律 芋 侵 拓 泉 袂 拭 祆 炭 頁 庭 陞 拇 泯 沔 南 枰 叟 映 胥 香 娜 砂 姚 屍 柔 思 怯 室 咦 疫 相 閂 牲 俠 泗 突 卸 畏 兗 施 庠 姨 陝 柝 恃 畎 玩 耍 帥 亭 泅 係 哉 柚 性 威 咸 省 剃 侮 咻 歪 信 屋 羿 眨 蚤沿柿怨沼畋狎則洶怏 俗 衍 幽 峙 禺 宣 囿 型 要 冑 炫 咫 注 爰 胃 柞 昨 咿 帟 泳 垣 奕 俎 宥 籽 姻 彥 怔 者 竽 炸 禹 俑 胄 怡 弈 柱 胤 炤 泱 拄 昭 勇 姿 疣 徉 芝 拙 星 昱 紂 油 祉 肘 陣 貞 音 殃 咨 姪 政 盈 啊 唉 埃 按 俺 案 氨 陰 盎 敖 芭 笆 耙 柏 派 班 般 舨 梆 旁 紡 豹 剝 砲 倍 俾 狽 畚 俸 秘 服 畢 紕 病 拼 玻 亳 埔 睬 財 倉 豺 倡 倀 倘 晁 宸 疹 陳 秤 乘 城 翅 持 蚩 恥 臭 畜 祝 芻 郵 純 肫 祠 脆 衰 厝 疸 玳 耽 訑 倒 島 洮 娣 砥 玷 凋 挑 爹 釘 洞 恫 凍 桐 蚪 鬥 哦 峨 峰 俯 芥 拾 掀 疾 砝 核 宮 涇 倣 訐 鬼 活 眩 肥 洱 恩 家 羔 姬 芳 肪 脊 砠 拱 函 倥 骨 粉 蚣 根 桁 俱 借 紛 剛 個 匪 舫 婀 莢 肺 鬲 躬 朗 罟 股 芬 烘 釜 娌 花 哥 屐 倌 恍 洹 狠 柩 害 飢 哩 洽 蚶 倦 陸 桓 訌 耿 倨 疽 桔 軒 唔 哭 娥 徑 洎 格 恢 芙 夏 祖 倚 耕 洪 隻 恐 拷 恪 哼 盍 航 挂 哽 恭 拮 矩 狡 浹 逕 氦 津 莖 耗 怪 唧 埂 括 烙 貢 肱 洌 珪 租 桂 高 倔 酒 桅 華 兼 缺 倪 記 晉 挾 砰 峻 桀 秦 桑 崁 候 祗 倆 校 烤 哨 峭 娟 恨 畔 們 凌 栗 豈 挈 拿 玲 倩 庫 料 涊 芹 痂 珀 倫 芰 洛 紐 眠 埋 秣 連 殺 框 氣 留 匿 務 耆 射 俳 疲 砌 師 偌 烈 敉 芟 娉 屘 哮 娑 能 皰 素 脅 洩 扇 訖 納 舐 殊 恤 破 迅 時 晒 桃 屑 閃 容 紗 恕 晌 神 晏 釙 畝 息 浬 衽 託 殉 拳 剖 十 弱 索 辱 拴 挺 奚 唐 倖 脩 書 徒 莧 虔 梃 耄 軔 洋 倭 洵 洫 娘 這 唆 臬 唏 栓 馬 翁 脈 烏 席 玆 育 起 迂 芽 配 氧 旅 朔 修 肢 胭 祟 邕 甜 洗 栩 娓 蚊 挖 訊 珊 益 圃 狹 拯 娠 紜 氤 訕 峴 秧 浥 峽 舀 疼 紙 窄 討 展 狩 砷 訓 迄 訏 孫 紊 蚜 紋 奘 條 衷 泰 殷 冢 芯 笑 恬 娛 拽 砸 特 致 套 針 耘 恙 洧 祚 徐 晅 唁 烊 窈 酌 祐 砟 洲 宵 圄 站 芷 症 峪 宴 原 栽 軏 真 值 員 宰 效 朕 株 准 蚓 芸 悅 恣 砧 莊 指 秩 栘 剜 釗 珍 袁 座 桌 挨 崖 庵 盒 菴 捌 敗 彬 絆 邦 崗 苞 胞 袍 被 埤 苯 笨 崩 逼 敝 婢 閉 匾 貶 彪 婊 舶 缽 捕 埠 彩 採 參 淺 曹 側 赦 涔 斜 釵 產 唱 常 徜 娼 悵 萇 巢 紹 趁 晨 淨 匙 豉 敕 笞 移 胝 蛇 崇 涌 舂 絀 處 啜 船 釧 唇 脣 瓷 疵 從 族 粗 戚 崔 浚 挫 袋 帶 蛋 啖 啗 啁 鳥 得 笛 隄 唸 涎 彫 釣 涉 窒 頂 動 兜 脰 堆 舵 棻 屝 訪 啡 進 瓠 浩 寂 貨 偽 梵 烽 紼 蛄 基 捍 梗 萊 毫 堊 販 婦 啞 返 匐 酣 梡 戛 趺 晦 隊 副 翎 訛 紱 珞 敢 苟 豚 軛 芾 符 過 階 寄 乾 渦 邢 胡 規 苛 婚 訣 健 琍 釩 桿 堝 渠 旌 狷 梨 率 凰 堅 掛 崑 剪 海 康 救 狼 胛 啃 梱 袈 患 痕 勘 淪 假 蛆 娶 貫 苦 釭 浸 焊 蚵 麥 婁 祭 許 硎 強 斛 頃 捫 雀 唬 狸 崛 浮 夠 粒 教 圈 涅 匏 苣 徠 胖 彗 烹 捎 苗 國 茄 捆 勒 那 袞 眷 票 區 皎 捐 悍 崆 曼 盔 畦 笠 茂 培 眶 略 涓 悔 掄 淒 鹿 浦 崙 卿 蚯 密 痊 婆 淚 宿 近 將 貧 悽 蛉 羚 苓 梢 啤 胚 旎 牽 聊 偶 扈 阡 犁 珮 覓 浪 捲 茉 梁 竟 啟 茅 冕 麻 釦 啪 徙 術 鹵 流 聆 售 偭 崎 悌 捂 婪 徘 苜 舷 毬 脤 訥 望 唳 氫 野 您 奢 孰 梂 研 浴 視 商 梧 捏 偏 眸 設 羞 若 敏 爽 悄 堂 粕 鄉 捨 寇 偎 悚 梅 挽 掃 胎 恿 梭 帷 紳 悟 梳 脫 絃 雪 英 偯 祥 苔 偷 痔 眼 敘 焉 眺 邪 苒 組 消 偵 現 萵 紲 茆 晚 紫 紮 笙 桶 雩 庶 悠 啕 訟 媧 屜 茁 停 剔 軟 袒 崤 晤 笮 貪 婉 捉 琊 袖 晝 梔 異 陽 窕 勗 做 域 趾 專 訢 械 梓 屠 酗 廷 倏 旋 唯 執 終 晞 涕 問 扎 崧 烯 習 偉 郾 梯 苑 振 庸 圉 責 斬 唾 延 悉 尉 唷 細 寅 章 痍 訝 敔 迎 帳 蛀 庾 猙 珠 蚱 堉 啄 張 偕 掙 硃 欲 梟 翌 偺 蛭 週 偃 眾 浙 崢 魚 捱 剴 傲 棍 跋 排 阪 鈑 棒 傍 彭 綁 堡 報 裒 悲 焙 邶 備 揹 賁 弼 幅 筆 詖 費 跛 博 番 發 跑 補 采 裁 猜 孱 殘 傖 草 策 廁 惻 測 曾 茶 接 喳 單 敞 猖 場 淌 脹 朝 超 鈔 掣 堪 棧 盛 程 啻 喜 喫 飭 惆 鈕 楮 淑 硫 絮 喘 窗 創 捶 椎 淳 荃 茨 嵯 措 茲 詞 淙 湊 淬 悴 皴 最 答 迭 達 棣 貸 淡 氮 悼 盜 登 等 第 堤 迪 邸 詆 惦 奠 蜓 掉 貂 跌 耋 喋 絰 軼 棟 湩 筒 痘 短 敦 防 貳 筏 鈍 斐 絕 焚 琺 楛 揮 飯 畫 渙 控 絞 淦 涪 壺 渾 惇 雋 喟 復 鈇 絡 貴 裂 棚 割 袱 茫 筋 傀 捧 扉 喝 廄 凱 掘 聒 戢 鄗 景 嗟 堠 蒐 喉 給 傅 皖 訶 傢 結 減 喊 富 黃 棘 焦 荒 窖 棺 間 混 馮 幾 悸 閎 猓 椒 皓 胱 胳 距 喀 愒 寒 迦 硬 集 痙 捩 啾 晷 杰 傑 揀 蛟 閒 鈐 棵 鈣 徨 詁 蛙 淮 蛤 惡 吭 惚 皋 喚 禍 厥 掬 塊 几 葷 賀 荐 嵌 換 街 阱 晶 帽 蛞 雇 辜 極 睏 惠 期 鉅 棄 棋 嵇 稈 蛐 裡 惑 鈞 描 評 喇 筐 違 買 就 茴 蛔 喬 荔 琉 竣 牌 媚 喱 戟 鈉 捻 晾 迫 淋 氯 邱 涵 量 貿 理 胼 開 茗 淇 窘 軻 絳 捺 荏 祿 莒 瑯 琅 惱 喪 勞 寐 掠 啦 鄒 嵐 述 涼 剩 普 深 欽 殼 掏 阮 啼 閔 情 森 跚 腎 邵 婿 翕 腆 崴 喃 涸 棉 清 稅 湯 然 涮 幃 黑 痞 無 斯 棲 粟 越 欺 飪 痢 硝 俟 稍 迢 瘦 啣 韌 球 毯 絨 散 疏 愜 耜 椅 殖 猛 茹 痛 推 巽 悻 跆 赧 覃 晰 絢 焜 媒 尋 善 授 舜 淞 硯 液 痠 喔 淫 棠 葦 惟 淅 猶 祁 晴 筍 淘 胸 喂 舒 須 項 跎 茸 探 塭 葉 圍 茱 惋 粥 悶 堯 廂 飧 徇 犀 壹 惜 順 迤 傘 絛 禽 絲 訴 淆 勝 淤 甥 閏 嫂 統 飩 添 胰 虛 惘 萬 掩 氬 酥 診 翔 荀 詐 稀 勛 雄 循 咱 棗 涯 淹 掌 寓 雯 証 湲 湧 掖 揚 運 詛 雁 飲 詔 蛛 智 爺 淵 哲 棕 喻 隕 替 遊 註 幀 喧 云 堰 甦 涿 淄 童 棹 詠 閑 婷 貯 象 馭 貼 軸 雅 茵 黍 雲 傚 釉 貽 痣 喲 植 媛 尊 脂 阿 矮 乃 愛 暗 鉗 骯 嗷 奧 媼 靶 湃 斑 頒 徬 刨 雹 鉋 飽 碑 陂 痺 迸 辟 荸 愎 嗶 媲 鉍 剽 摒 稟 脖 渤 募 鈸 鉑 鈽 踩 滄 蒼 插 搽 捷 荼 詫 搓 剷 腸 剿 勦 琛 填 湛 嗆 搶 誠 嗤 提 莉 飾 馳 傭 愁 稠 酬 楚 跦 揣 傳 惴 愴 蠢 揍 催 瘁 搭 塔 靼 塌 迨 遞 馱 詹 誕 當 搗 荻 滌 殿 碘 鈿 電 碉 跳 牒 鼎 渡 堵 督 塗 睪 頓 碓 追 節 渴 港 跡 概 跺 感 莞 琨 貉 蛾 該 陀 詻 躲 詬 煩 惶 稞 痱 湟 溝 漣 畸 揩 嗑 琿 際 勣 跤 揭 豢 經 僇 雊 舅 蓋 楓 閘 賅 嗯 鉀 荷 愍 惰 猴 會 煌 湖 愕 摑 郃 愣 睢 煥 郝 戡 幹 蓮 筠 賈 蜂 嫁 彙 揖 詼 愾 鉤 溉 喙 楷 莆 琯 琥 稜 匯 解 較 詰 嫉 迥 頑 慨 跟 靖 綑 揪 郊 鈷 筧 暇 髡 湔 暉 腳 絹 逅 硿 斡 號 酪 毽 晃 話 亂 嘩 詭 雉 僅 湣 幌 脯 裘 睛 愀 廉 鳩 貊 夢 裊 煎 窟 粱 愷 楫 麂 廊 渺 賃 煉 粳 毀 禁 零 夸 硼 蒞 祺 揆 盟 箝 酩 廓 娩 雷 楠 琴 睫 瓶 賂 琳 琦 誇 楞 莓 敬 跪 睞 游 詩 軾 賄 睥 睨 湄 嗥 聘 搆 窠 慄 渥 傾 睦 逆 援 揉 蜊 莽 滅 鉚 煤 蓀 惹 嗎 御 勤 迷 路 媽 鈴 蜆 聖 楣 楝 腦 琵 痳 煆 煨 煞 塢 愆 虜 弒 煖 稔 塞 鈹 蛻 歲 痲 鼠 勢 遜 群 獅 綏 搔 詮 鉛 塑 琪 試 痿 暑 蜈 嗣 嗜 筮 握 搜 猩 頌 飼 熄 蜀 愉 嗓 睡 逃 筵 裙 詨 損 嗇 艇 肆 溼 莘 送 裟 農 碗 肅 馴 莫 湍 莎 傷 頊 遙 煙 渭 預 義 煦 暖 嫌 熙 嗉 琶 痰 窩 想 歇 輊 嗡 業 蓆 遝 廈 貲 覜 佣 暍 碎 靴 訾 塘 莠 塒 溫 蔭 渲 衙 羨 嵩 暄 楔 嗚 嗦 裝 蛹 嗅 罩 裕 媳 慍 鉉 蝓 微 渣 照 猥 飴 湘 詢 圓 搖 退 蜃 楊 傯 惺 暈 載 賊 煬 資 斟 諍 腫 詳 肄 琢 準 稚 阻 塋 椰 睜 裔 愚 煜 榆 鈾 與 暘 盞 意 新 楨 虞 湮 郁 雍 詣 塚 渚 遠 蜇 楹 罪 園 誅 渝 瘀 煮 孳 粵 搾 禎 置 債 愈 猷 腌 閡 銨 熬 搬 榜 嘐 悖 菩 鼻 碧 弊 裨 幣 箄 脾 嫖 彆 賓 鉼 餅 菠 搏 箔 蔔 駁 僕 溥 誧 菜 綵 慘 慚 臧 嘈 造 僧 察 團 嶄 漸 裳 嫦 嘗 暢 綽 塵 稱 逞 槍 趟 郢 滯 僮 種 溴 綢 裯 蔥 箠 綴 慈 雌 滋 翠 粹 萃 瘩 瘍 凳 鄧 嫡 翟 嘀 逐 適 滇 墊 蜩 趙 碟 逗 睹 端 對 餌 綞 孵 菲 箋 閨 嘎 幗 奪 菔 構 罰 愧 鳳 腐 福 餃 齊 鼓 管 降 爾 複 歌 墮 腑 腓 逢 漓 廣 菰 蝕 搞 廖 奩 趕 僭 滑 監 嘉 榦 划 酷 兢 暨 酵 裹 閥 裴 跼 蔣 誥 嫘 菽 匱 僱 蝸 豪 犒 菅 僑 瘋 郡 盡 甄 瘓 膏 誨 碣 菱 寡 聚 樺 綾 翡 綸 箕 竭 滬 睽 蜻 槁 頗 菁 逛 菊 綱 槐 溘 誡 嗨 銬 閣 犖 猾 箇 褂 骷 厲 漢 僥 菌 瑚 瑙 骰 慣 劃 瑁 閤 截 鉸 塹 腔 槓 鄰 僚 赫 寞 輕 寬 陌 寥 輔 瑪 雒 鉻 屢 熒 么 滲 榴 蜜 幕 辣 碰 寧 筷 槨 逵 鳴 精 魂 夥 裸 誑 嘛 魁 鋁 嫩 髦 滿 膂 郎 瞄 颯 榮 冥 麼 嶇 誤 瘧 溜 緊 閭 貍 綠 蜷 蔆 閩 疑 領 暱 溺 伙 需 陋 榷 綺 認 劂 滷 滾 蜢 透 墓 限 墅 榔 嫗 仆 境 寢 慝 耶 榕 萌 嘔 暝 溧 槃 瑞 幔 煽 蜴 榣 通 綿 摟 銘 瑟 碳 搴 敲 署 塾 誚 酸 嗾 塽 歉 誓 褕 榻 算 嘍 萋 惕 瘟 漬 賒 綬 聞 慟 台 餉 瞇 輓 腕 鞋 慎 隨 寨 菟 臺 熔 壽 菸 寤 說 嗽 銜 網 銑 綰 溶 逝 嫣 溢 颱 滂 萍 旗 緒 碩 瘉 腴 僩 舔 溪 舞 瑤 瑣 萄 銓 途 劓 維 綜 製 蜿 嘆 實 速 艋 榫 墟 榭 瑕 蜥 誣 綻 搪 銅 鞅 氳 養 鄭 誦 逍 帚 窪 酴 慇 旖 冤 志 厭 萎 腋 逖 態 像 銖 溯 毓 僎 獄 撾 榨 肇 漁 鳶 瑛 源 滕 漲 幛 夤 蜘 萸 圖 瑜 愿 輒 語 箏 粽 緇 墜 賑 嘖 韶 錚 滔 銀 僖 誘 摭 猿 誌 熊 滓 彰 榛 皚 鞍 醃 噢 澆 罷 瘢 暴 褒 褓 葆 葡 輩 鋇 奔 陛 陴 蝙 褊 編 標 漂 魄 撥 潑 部 撲 槽 漕 增 層 嬋 憚 摻 諂 廠 嘮 徹 瞋 徵 撐 踟 墀 遲 齒 樁 衝 除 廚 著 褚 諸 鋤 幢 瘡 醇 輟 磁 賜 樅 趣 醋 摧 磋 銼 褡 逮 彈 撢 儅 蕩 稻 導 德 嶝 滴 蒂 敵 締 調 蝶 踢 董 陡 嘟 樞 賭 緞 墦 廢 敷 銳 范 橢 輥 樊 劍 盥 膚 範 墳 憤 葛 駕 碾 賡 餓 墩 髮 誹 幡 戮 蝌 銲 撫 踡 葭 劇 踝 毆 穀 磕 獎 澗 慧 瘠 緘 颳 儉 橄 蝠 頜 撈 稷 犛 篁 緝 蝦 麩 鋰 潰 葫 駙 腱 稿 駒 郭 落 磊 慷 羯 鴃 緩 槳 糊 嘰 賤 頫 稽 險 稼 賦 噴 價 蝴 嘿 膠 腹 劊 漿 鋒 麾 箭 鞏 寮 頡 邁 篌 箴 模 慮 畿 瑰 嬌 挪 摩 款 瞌 熟 嘹 課 儈 論 輪 窮 瑾 漏 諒 慕 僵 噗 輦 樓 漠 貓 樑 褐 輛 踐 踫 澠 憩 蝗 樂 面 磐 褲 瞑 葩 憐 碼 摺 凜 憫 慶 劉 噙 審 駛 翩 窯 黎 歐 諾 輝 遼 罵 墨 靠 練 鬧 潛 緙 駑 葵 暮 鋪 慼 請 踞 髯 撓 履 霉 賠 潔 廟 漫 蝨 魯 熱 嶔 嫻 噎 撳 盤 誶 慢 瘤 緯 銻 豎 腿 冪 僻 魅 奭 選 霄 賣 餒 貌 撻 摸 寫 確 殤 劈 緲 漆 甌 勰 耦 慫 數 輞 摔 慰 誰 潤 衛 嬈 傻 腰 嬉 賞 褪 艘 遺 震 餘 緬 曄 腥 槭 億 瑭 腮 諛 諄 噓 遷 駝 征 陷 漱 嘶 踏 廝 潯 歎 篇 儂 鴆 躺 蕪 鋅 諉 撚 憮 鴉 霆 漪 憂 箸 潠 豌 逶 慾 儀 駟 緹 嫵 閱 賢 腺 談 線 嘻 箱 熨 逸 慵 噪 院 銷 皺 漩 毅 牖 質 誼 醉 鄴 樟 緣 暫 駐 漳 瞎 萱 幟 豬 獐 瑩 陬 漾 賬 輜 潟 樣 摯 影 魷 演 葬 摘噫 噯 諳 懊 澳 聱 辦 辨 磅 螃 鮑 蓓 憊 壁 篦 瞥 遍 錶 撇 儐 頻 播 膊 潘 蒲 餐 艙 糙 儕 潺 禪 氅 潮 縐 撤 澈 橙 縝 諶 澄 醒 褫 嘯 熾 憧 潼 儔 醜 撮 踹 陲 錘 踱 輳 錯 璀 蹉 營 壇 擔 潭 噹 擋 燙 道 陶 燈 諦 蹄 靛 澱 雕 諜 錠 都 逾 獨 篤 遁 噸 燉 霏 縫 濩 靜 擭 蒿 澤 薦 鄂 鋼 縉 澴 暸 遏 檠 餞 噶 墾 獲 錮 萼 霍 獗 輯 翰 翮 骸 冀 鬨 鍵 奮 陵 橫 舉 劑 駭 諷 諫 撿 錢 寰 璟 橋 磬 餛 勵 篙 縞 頷 還 駱 憬 璣 薑 噩 頸 橇 據 鴣 慌 學 機 懍 膈 豭 噤 髻 骼 撩 衡 儘 鍋 薊 龜 瞰 館 燎 糕 罹 橘 噱 錦 彊 錕 縛 鋸 遑 謊 瞞 噥 曆 扑 膨 龍 潦 縑 螂 燐 積 遇 褸 璘 輻 膩 隍 黔 薔 擄 朴 嬝 謎 陪 穋 擂 霓 錨 賴 錳 糢 諱 澎 窺 璃 濃 錄 瘺 餡 篛 睿 螟 燃 駢 燜 憔 霖 憑 盧 撕 垮 錡 濛 瞟 頰 褥 瓢 鞘 橈 歙 靦 簑 撬 燒 穆 親 瘸 蒙 螞 噬 撒 曉 歷 樵 磧 熹 凝 篩 錫 默 頭 膳 霎 磨 蓉 橡 器 樸 撰 擒 儒 諧 遛 融 興 謀 潸 憲 鴒 羲 謂 鴦 蹂 壅 霑 嶼 蒜 蓊 蓄 鴕 燄 勳 頹 曇 薩 輸 豫 遂 篠 筑 璋 諺 擁 磚 螢 焰 澦 蓑 諭 縣 築 穌 濁 餚 赭 錐 憶 戰 隱 頤 穎 嬴 嘴 擇 撙 燠 樹 緻 撞 整 鴛 蕭 臻 鴨 謁 諼 閻 憎 蒸 遐 燕 錙 髭 踵 樽 篆 踴 糖 縈 瘴 縊 覦 諮 癌 隘 曖 璦 遨 襖 豳 闆 幫 謗 臂 繃 蔽 斃 檗 蹕 濱 擘 擦 蔡 燦 操 澡 澶 毚 償 嚐 濤 瞠 騁 鯈 黜 儲 歜 鎚 簇 薺 縱 總 聰 趨 黛 澹 憾 檐 膽 檔 盪 蹈 瞪 磴 點 褶 懂 鍍 鍛 邇 糞 獷 谿 蟠 繁 簍 鍔 磺 鍰 燭 隋 環 艱 媾 屨 禮 檢 鮭 豁 璜 儡 濫 講 戲 激 駿 闋 顆 尷 轄 礁 轂 鞠 虧 鍊 爵 燴 覬 壑 磯 臉 檜 嚎 壕 鮫 壙 癘 撼 擱 鴿 澧 蟈 購 徽 懇 蔑 擊 賻 颶 燬 糜 瞭 餽 縷 鎂 縲 膿 闌 嶺 矯 療 臨 績 縵 魎 擠 擴 闊 嚅 罄 舊 濟 濬 輾 縹 磷 繆 璞 嚇 摹 韓 隆 膾 聯 獰 隸 謨 螫 嬪 鴻 篾 藍 繅 濂 糠 橾 譁 擬 縮 髁 繈 癆 鍬 彌 斂 鍥 懋 賽 鼾 襄 麋 鮪 嶸 甍 嬤 謄 螻 蔓 嚀 膜 蟋 黏 懈 聲 擎 燥 檀 濘 謙 螺 糝 遢 燮 遣 孺 澀 蟒 殮 擅 闈 褽 縿 瞧 霞 檣 縴 擰 濕 曙 謐 霜 禧 瞳 瞬 膛 蓿 牆 餵 縯 雖 甕 蹊 螳 臀 蔚 檄 蟀 鮮 贅 嚏 蓬 簌 聳 濰 壓 櫛 賸 嚮 應 黝 蹋 蟑 翼 餿 謝 翳 醣 氈 優 謠 褻 隅 燧 壎 蔗 鍾 轅 嶽 輿 擲 嬰 膺 賺 歟 醞 禦 齋 膝 糟 鍚 翱 擺 鎊 蹦 鄙 璧 馥 鞭 邊 殯 臏 檳 蕃 璨 繒 蟬 瀋 闖 鎗 痴 抬 離 虫 蟲 檮 礎 雛 槌 戳 叢 藪 蹙 竄 戴 簞 襠 燾 禱 題 鼕 瀆 斷 顎 翻 鎬 瞿 簧 濺 檻 蕙 繙 隔 額 豐 鵠 遘 覲 覆 鎘 濡 丰 藥 燼 雞 闕 犄 簡 壘 鯀 鵝 曠 穢 醬 鯽 歸 翹 蕞 濠 騎 嚕 穫 蹟 櫃 謹 闔 瞽 簣 朦 瞼 燸 繭 蕊 餾 攆 癖 霤 鎳 竅 擾 臍 蕉 懦 釐 蹣 謬 鵑 繚 繩 蟯 濮 櫚 濾 蕨 鯉 檸 瀏 韹 矇 糧 繞 瓊 鏈 霧 穠 獵 鎮 鞣 鬆 鞦 穡 濯 松 雙 璩 蕈 繕 聶 職 鎖 鎔 擻 瀉 藷 鯊 顏 櫂 檯 鎢 曜 懣 耨 蔬 魍 餮 穗 嬸 魏 軀 觴 瞻 謫 闐 璿 颺 簪 鄘 黠 鎰 罈 隙 鄞 颼 醫 雜 鬃 障 擢 彝 轉 蹤 遮 癒 顓 蠅 擷 燻 遭 織 藝 鼬 礙 藹 鵪 鏖 薜 瓣 薄 爆 瀑 簿 曝 襞 鏢 癟 瀕 簸 蟾 儳 鏟 鯧 嘲 帘 懲 癡 疇 櫥 鶉 鏃 辭 蹴 蹲 躂 蹬 鏑 顛 鯛 胴 櫝 牘 犢 饉 蹶 蟻 躉 壞 醮 懶 類 疆 鏗 遴 譎 麗 繪 羹 鏡 鯨 關 轎 瀨 繳 繫 隴 醱 系 薨 鵬 襤 蘆 懷 鄱 鯖 羸 藺 譜 爍 攀 鏍 瀝 麒 胯 靡 瀘 麓 鬍 襟 譏 轔 廬 臘 難 懵 坏 蕾 簽 礦 饅 簾 鏝 蟆 識 蘋 湎 壢 櫓 鏤 獺 譙 礪 獸 蹼 鵲 蹺 薯 羅 癢 繹 韜 麴 鵡 穩 蘇 鏘 蘊 勸 騖 譆 鏜 羶 璽 蟹 願 嚥 臃 譚 薇"; //不重复的汉字
                fontStr = fontStr.replace(/\s*/g, "");
                var fontChars = [];

                var avg = Math.floor(
                    parseInt(this.imgWidth) / (parseInt(this.defaultNum) + 1)
                );
                var tmp_index = "";

                for (var i = 1; i <= this.defaultNum; i++) {
                    fontChars[i - 1] = this.getChars(fontStr, fontChars);

                    tmp_index = Math.floor(Math.random() * 3);
                    ctx.font = fontSizeArr[tmp_index];
                    ctx.fillStyle = this.rColor();

                    if (Math.floor(Math.random() * 2) == 1) {
                        var tmp_y =
                            Math.floor(parseInt(this.imgHeight) / 2) +
                            tmp_index * 20 +
                            20;
                    } else {
                        var tmp_y =
                            Math.floor(parseInt(this.imgHeight) / 2) - tmp_index * 20;
                    }

                    ctx.fillText(fontChars[i - 1], avg * i, tmp_y);
                    this.fontPos[i - 1] = {char: fontChars[i - 1], x: avg * i, y: tmp_y};
                }

                for (var i = 0; i < this.defaultNum - this.checkNum; i++) {
                    this.shuffle(this.fontPos).pop();
                }

                var msgStr = "";
                for (var i = 0; i < this.fontPos.length; i++) {
                    msgStr += "\xa0\xa0" + '"' + this.fontPos[i].char + '"' + "\xa0\xa0";
                }

                this.text = msgStr.substring(0, msgStr.length - 1);

                return this.fontPos;
            },
            //获取坐标
            getMousePos: function (obj, e) {
                var x = e.offsetX - 5;
                var y = e.offsetY - 5;

                return {x, y};
            },
            //递归去重
            getChars: function (fontStr, fontChars) {
                var tmp_rand = parseInt(Math.floor(Math.random() * fontStr.length));
                if (tmp_rand > 0) {
                    tmp_rand = tmp_rand - 1;
                }
                var tmp_char = fontStr.charAt(tmp_rand);
                if (fontChars.indexOf(tmp_char) == -1) {
                    return tmp_char;
                } else {
                    return this.getChars(fontStr, fontChars);
                }
            },
            //洗牌数组
            shuffle: function (arr) {
                var m = arr.length,
                    i;
                var tmpF;
                while (m) {
                    i = (Math.random() * m--) >>> 0;
                    tmpF = arr[m];
                    arr[m] = arr[i];
                    arr[i] = tmpF;
                    //[arr[m], arr[i]] = [arr[i], arr[m]];	//低版本浏览器不支持此写法
                }
                return arr;
            },
            //创建坐标点
            createPoint: function (pos) {
                this.tempPoints.push(Object.assign({}, pos));
                return ++this.num;
            },
            comparePos: function (fontPos, checkPosArr) {
                var flag = true;
                for (var i = 0; i < fontPos.length; i++) {
                    if (
                        !(
                            parseInt(checkPosArr[i].x) + 40 > fontPos[i].x &&
                            parseInt(checkPosArr[i].x) - 40 < fontPos[i].x &&
                            parseInt(checkPosArr[i].y) + 40 > fontPos[i].y &&
                            parseInt(checkPosArr[i].y) - 40 < fontPos[i].y
                        )
                    ) {
                        flag = false;
                        break;
                    }
                }
                return flag;
            },
            refresh: function () {
                this.$parent.$emit('refresh')
                this.spanshow = true;
                this.tempPoints.splice(0, this.tempPoints.length);
                this.barAreaColor = 1;
                this.barAreaBorderColor = "#ddd";
                this.bindingClick = true;
                this.fontPos.splice(0, this.fontPos.length);
                this.checkPosArr.splice(0, this.checkPosArr.length);
                this.num = 1;
                this.imgRand = Math.floor(Math.random() * this.imgName.length); //随机的背景图片
                var img = new Image();
                img.src = this.getRandomImgSrc();
                // 加载完成开始绘制
                var _this = this;
                img.onload = function (e) {
                    _this.$nextTick(() => {
                        console.log('_this.fontPos',_this.fontPos);
                        _this.fontPos = _this.drawImg(_this.$el, this);
                    })
                }
                this.showRefresh = true;
            },
        },
        watch: {
            // type变化则全面刷新
        },
        mounted() {
            // 禁止拖拽
            this.$el.onselectstart = function () {
                return false;
            };
            this.init()
        },
        i18n: {
            messages: {
                "en-US": {},
                "zh-CN": {},
            },
        },
    };
</script>
<style lang="less" scoped="scoped">
    .verify-code {
        font-size: 20px;
        text-align: center;
        cursor: pointer;
        border: 1px solid #ddd;
    }

    .cerify-code-panel {
        height: 100%;
        overflow: hidden;
    }

    .verify-code-area {
        float: left;
    }

    .verify-input-area {
        float: left;
        width: 60%;
        padding-right: 10px;
    }

    .verify-change-area {
        line-height: 30px;
        float: left;
    }

    .varify-input-code {
        display: inline-block;
        width: 100%;
        height: 25px;
    }

    .verify-change-code {
        color: #337ab7;
        cursor: pointer;
    }

    .verify-btn {
        width: 200px;
        height: 30px;
        background-color: #337ab7;
        color: #ffffff;
        border: none;
        margin-top: 10px;
    }

    .zkr {
        .verify-bar-area {
            display: flex;
            justify-content: space-evenly;
            align-items: center;
            position: relative;
            background: #ffffff;
            -webkit-box-sizing: content-box;
            -moz-box-sizing: content-box;
            box-sizing: content-box;
            -webkit-border-radius: 4px;

            /deep/ .el-input__icon {
                display: inline-block;
            }

            @media screen and (max-width: 1800px) {
                /deep/ .el-input__icon {
                    display: inline-block;
                    height: 40px;
                }
            }

            .pass {
                /*background: url(../../../../static/img/login/4.png) no-repeat center center;*/
            }

            .verify-msg {
                display: flex;
                align-items: center;
                color: rgb(145, 152, 163);
                line-height: 40px;

                .size {
                    font-size: 12px;
                }

                .fontColor {
                    color: rgb(145, 152, 163);
                    margin-bottom: 0px;
                }

                .fontColor1 {
                    color: #d9534f;
                    margin-bottom: 0px;
                }

                .fontColor2 {
                    color: #4cae4c;
                    margin-bottom: 0px;
                }

                > p {
                    font-size: 18px;
                }
            }

            .refresh {
                color: rgb(25, 136, 252);
                cursor: pointer;
                line-height: 50px;
            }
        }
    }

    .verify-bar-area .verify-move-block {
        position: absolute;
        top: 0px;
        left: 0;
        background: #fff;
        cursor: pointer;
        -webkit-box-sizing: content-box;
        -moz-box-sizing: content-box;
        box-sizing: content-box;
        box-shadow: 0 0 2px #888888;
        -webkit-border-radius: 1px;
    }

    .verify-bar-area .verify-move-block:hover {
        background-color: #337ab7;
        color: #ffffff;
    }

    .verify-bar-area .verify-left-bar {
        position: absolute;
        top: -1px;
        left: -1px;
        background: #f0fff0;
        cursor: pointer;
        -webkit-box-sizing: content-box;
        -moz-box-sizing: content-box;
        box-sizing: content-box;
        border: 1px solid #ddd;
    }

    .verify-img-panel {
        margin: 0;
        -webkit-box-sizing: content-box;
        -moz-box-sizing: content-box;
        box-sizing: content-box;
        border-radius: 3px;
        position: relative;
    }

    .verify-img-panel .verify-refresh {
        width: 25px;
        height: 25px;
        text-align: center;
        padding: 5px;
        cursor: pointer;
        position: absolute;
        top: 0;
        right: 0;
        z-index: 2;
    }

    .verify-img-panel .icon-refresh {
        font-size: 20px;
        color: #fff;
    }

    .verify-img-panel .verify-gap {
        background-color: #fff;
        position: relative;
        z-index: 2;
        border: 1px solid #fff;
    }

    .verify-bar-area .verify-move-block .verify-sub-block {
        position: absolute;
        text-align: center;
        z-index: 3;
        border: 1px solid #fff;
    }

    .verify-bar-area .verify-move-block .verify-icon {
        font-size: 18px;
    }

    .verify-bar-area .verify-msg {
        z-index: 3;
    }
</style>
