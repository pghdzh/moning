<template>
  <div class="morning-page" :class="{ 'reduced-motion': reducedMotion }">
    <!-- 背景机械纹理层 -->
    <div class="grid-bg"></div>
    <div class="hex-bg"></div>

    <!-- 动态光晕层 -->
    <div class="glow-orb glow-1"></div>
    <div class="glow-orb glow-2"></div>
    <div class="glow-orb glow-3"></div>

    <!-- 扫描线 & 数据流 & 其他装饰 -->
    <div class="scan-line"></div>
    <div class="data-flow"></div>
    <div class="red-dot red-dot-1"></div>
    <div class="red-dot red-dot-2"></div>
    <div class="red-dot red-dot-3"></div>
    <div class="red-dot red-dot-4"></div>
    <div class="mech-corner"></div>

    <!-- 新增：旋转数据环 & 闪烁粒子 & 透明腿特效 -->
    <div class="data-ring"></div>
    <div class="floating-particles">
      <span v-for="n in 12" :key="n" class="particle"></span>
    </div>
    <div class="leg-glow"></div>

    <main class="container" ref="mainContainer">
      <!-- ========== HERO 区域 ========== -->
      <header class="hero" role="banner">
        <div class="left-col">
          <div class="avatar-card">
            <div class="avatar-glow"></div>
            <picture class="avatar-wrap">
              <img
                src="@/assets/images1/moning-slty (14).webp"
                class="avatar"
                loading="lazy"
                decoding="async"
                alt="莫宁"
              />
            </picture>
            <div class="avatar-meta">
              <span class="avatar-badge">共鸣能力：星枢演构</span>
              <span class="avatar-badge">深空联合研究院</span>
            </div>
          </div>
        </div>
        <div class="right-col" ref="metaCard">
          <div class="hero-panel">
            <h1 class="name">
              莫宁
              <span class="jp">Mornye · 星炬学院教授</span>
            </h1>
            <p class="subtitle">
              「使用人造的双腿踏足大地，而心中是璀璨群星。」
            </p>
            <p class="lead">
              一切规律皆可被观测，一切现象皆可被析解，正如理想远在光年之外，抵达的方法却在双手咫尺之间。
            </p>
            <div class="hero-stats">
              <div class="stat-item"><span>武器</span> 长刃</div>
              <div class="stat-item"><span>属性</span> 热熔</div>
              <div class="stat-item"><span>定位</span> 治疗 · 辅助</div>
              <div class="stat-item"><span>CV</span> 佟心竹 / 石见舞菜香</div>
            </div>
          </div>
        </div>
      </header>

      <!-- ========== 战斗解析 - 核心机制 ========== -->
      <section id="personality" class="card personality">
        <div class="card-head">
          <h2>战斗解析 · 星枢演构</h2>
        </div>
        <ul class="traits">
          <li>
            <span class="trait-highlight">谐振场</span
            >：展开谐振场提供治疗和偏谐值累积效率加成。
          </li>
          <li>
            <span class="trait-highlight">干涉标记</span
            >：为目标附加【干涉标记】使其受到的伤害提升。
          </li>
          <li>
            <span class="trait-highlight">延奏技能</span
            >：为队伍中的角色提供全伤害加深。
          </li>
          <li>
            <span class="trait-highlight">双能量系统</span
            >：普攻命中目标可以回复【静质量能】；处于广域观测模式时，普攻和共鸣技能命中目标可以回复【相对动能】。
          </li>
          <li class="mt-2">
            <span class="trait-highlight">核心流程</span
            >：EA-AA-闪-Z-E-AAA-Z-R-Q（普攻=A，重击=Z，共鸣技能=E，共鸣解放=R，声骸=Q）
          </li>
        </ul>
      </section>
      <section id="resonance-status" class="card overview">
        <div class="card-head">
          <h2>共鸣状况 · 深空联合评估报告</h2>
        </div>
        <div class="two-col reports-grid">
          <!-- 频谱检验报告卡片 -->
          <div class="report-card">
            <div class="report-header">
              <span class="report-icon">📊</span>
              <h3 class="section-subtitle">频谱检验报告</h3>
            </div>
            <div class="report-content">
              <div class="report-section">
                <h4>1. 共鸣能力评估</h4>
                <p>
                  <span class="badge">自然型共鸣者</span> 声痕位置：左侧大腿外侧
                </p>
                <div class="spec-item">
                  <span class="spec-label">作用范围：</span>
                  <span class="spec-value"
                    >能力有效作用半径约为20米。针对范围内所有具备精密内部结构的机械体，表现为对目标内部元件的直接、精细化操控。操控精度在有效范围内保持恒定，未观察到随距离增加而衰减的现象。</span
                  >
                </div>
                <div class="spec-item">
                  <span class="spec-label">生效前提：</span>
                  <span class="spec-value"
                    >能力的成功发动，依赖测试人对目标机械内部结构的完全解析。对于结构未知的物体，能力无法生效。</span
                  >
                </div>
                <div class="note">
                  “等等，这一条是怎么回事……我从来没有见过莫宁教授操控不了的机械啊？！”
                </div>
              </div>
              <div class="report-section">
                <h4>2. 身体状况说明</h4>
                <p>
                  <span class="badge">诊断：</span
                  >测试人患有先天性腰骶神经发育不全，临床表现为尾椎以下运动功能永久性丧失，无自主行动能力。
                </p>
                <p>
                  通过定制型神经接入式义肢系统，已实现完全自主行走与日常活动能力，多年期间表现稳定，本次评估为常规状态复查。
                </p>
              </div>
            </div>
          </div>

          <!-- 超频诊断报告卡片 -->
          <div class="report-card">
            <div class="report-header">
              <span class="report-icon">⚡</span>
              <h3 class="section-subtitle">超频诊断报告</h3>
            </div>
            <div class="report-content">
              <div class="report-section">
                <h4>波形分析</h4>
                <p>
                  受试样本波形检测呈现椭圆形波动，时域表示稳定，已恢复稳定，检测结果判断为正常。
                </p>
              </div>
              <div class="report-section">
                <h4>诊断结果</h4>
                <p>
                  <span class="badge success">正常</span>
                  超频临界值正常，稳定性高，暂无超频风险。
                </p>
                <p><span class="badge warn">过往超频史</span> 最高等级：轻</p>
                <p class="history-note">
                  主诉于数年前因维持大型信号塔运作之需，曾主动引发短暂超频。因及时救治，未观察到超频相关后遗症。
                </p>
              </div>
              <div class="report-footer">
                <p class="quote">
                  「正因为能看到星星，人类才会对天空之外的世界产生好奇。」<br />——
                  那位前辈的话语，成为莫宁毕生追寻的起点。
                </p>
              </div>
            </div>
          </div>
        </div>
      </section>
      <!-- ========== 随身物品 - 记忆的锚点（保留图片） ========== -->
      <section id="items" class="card items-section">
        <div class="card-head">
          <h2>随身物品 · 记忆的锚点</h2>
        </div>
        <div class="items-grid">
          <div class="item-card" v-for="(item, idx) in items" :key="idx">
            <div class="item-image" v-if="item.img">
              <img :src="item.img" :alt="item.title" loading="lazy" />
            </div>
            <h3>{{ item.title }}</h3>
            <p>{{ item.desc }}</p>
          </div>
        </div>
      </section>

      <!-- ========== 人物剧情 · 莫宁纪事 ========== -->
      <section class="stories-section" ref="storyRef">
        <div class="card">
          <header class="card-head">
            <h2>莫宁纪事 · 从轮椅到群星</h2>
          </header>

          <main class="story-content">
            <transition name="fade-slide" mode="out-in">
              <div :key="current" class="story-item">
                <h3>{{ stories[current].title }}</h3>
                <p>{{ stories[current].desc }}</p>
              </div>
            </transition>
          </main>

          <footer class="story-nav">
            <button class="nav-btn" @click="prevStory">‹</button>
            <div class="pager">{{ current + 1 }} / {{ stories.length }}</div>
            <button class="nav-btn" @click="nextStory">›</button>
          </footer>
        </div>
      </section>

      <section id="timeline" class="card timeline-section">
        <div class="card-head">
          <h2>共鸣链 · 星枢六境</h2>
        </div>
        <div class="timeline-container">
          <div class="timeline-line"></div>
          <div
            v-for="(event, idx) in timelineEvents"
            :key="idx"
            class="timeline-item"
            :class="{ 'timeline-right': idx % 2 === 1 }"
            :style="{ animationDelay: idx * 0.15 + 's' }"
          >
            <div class="timeline-marker">
              <span class="marker-dot"></span>
            </div>
            <div class="timeline-card">
              <h4>{{ event.title }}</h4>
              <p>{{ event.desc }}</p>
            </div>
          </div>
        </div>
      </section>

      <section id="chains" class="card chains-section">
        <div class="card-head">
          <h2>彩蛋 · 星空的回响</h2>
        </div>
        <div class="chains-grid">
          <div
            v-for="(chain, idx) in chains"
            :key="idx"
            class="chain-card"
            :style="{ animationDelay: idx * 0.1 + 's' }"
          >
            <div class="chain-number">{{ idx + 1 }}</div>
            <h4>{{ chain.title }}</h4>
            <p>{{ chain.desc }}</p>
          </div>
        </div>
      </section>
    </main>

    <!-- 浮动小人 - 保留 -->
    <div class="floating-chibis">
      <img
        v-for="(pet, i) in chibiList"
        :key="i"
        :src="pet.src"
        :style="{ top: pet.top + 'px', left: pet.left + 'px' }"
        class="chibi-img"
        alt=""
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

// ========== 数据定义 ==========
// 物品数据（保留图片）
const items = [
  {
    img: new URL("./img/item1.png", import.meta.url).href,
    title: "改良星栈",
    desc: `深空联合每位研究员都需要佩戴的「星栈」装置，用于实时接收、监测学员的频率信号，以快速判断学生情况。

莫宁在原有功能的基础上，亲自对它进行了改造，使它能储存自己的共鸣能力并代为释放。这样，她就可以在思考问题的同时，用共鸣能力监控义肢状态、回复工作邮件、自动驾驶载具、让浮游炮运送营养剂……

总之，效率越高越好。`,
  },
  {
    img: new URL("./img/item2.png", import.meta.url).href,
    title: "旧学生证",
    desc: `莫宁在星炬学院就读时使用的学生证。

众所周知，星炬学院是全索拉里斯顶尖的人才培养机构。彼时轮椅之上的莫宁，畏惧社交，独来独往，将自己困于小房间里的纸张与理论中。星炬学院的入学邀请让她第一次看到一个可能性——即使无法行动的自己，在某处，也能有自己的价值。

那里一定有一个更为广阔的世界。`,
  },
  {
    img: new URL("./img/item3.png", import.meta.url).href,
    title: "「目光」",
    desc: `“头发总是垂下来的话，小心视力变差哦。”

因某位前辈的提醒，莫宁难得离开了实验室，去她极少踏足的集市买回这样两枚发卡。小小的发卡造型简单，功能上也只够把她厚重的刘海堪堪夹起，但看向镜中的自己时，她却感到一种忐忑又陌生的心情——仿佛整片视野，都正期待着一道目光的来临。

如今，她已经习惯于将那些遮挡视野的过长碎发一刀剪去，但这两枚不再有任何功能的发卡，依然被她别在发丝之间。`,
  },
  {
    img: new URL("./img/food.png", import.meta.url).href,
    title: "罐装效率",
    desc: `莫宁学生时代在料理课上完成的作品。省去了顶部为美观而放置的水果装饰，转而加入大量黄昏蜜罐，有效提高了单口能量补充效率。据那堂课上的同学所说，莫宁为了营养综合性，还尝试过寻找鱼肉与湿蕨团加入其中……`,
  },
];

// 故事数据（整合剧情）
const stories = [
  {
    title: "响声",
    desc: `莫宁从很小的时候就知道，自己是会给别人带来麻烦的。

家人在大多数时候都待她很好。母亲辞工在家看护她，父亲则为支付医药与检查的高昂费用，忙于工作鲜少归家。在那些记忆尚显模糊的日子里，母亲常会把她抱在怀里，说些柔软的音节，再将她放回那架属于她的小小床榻。某年生日，父亲难得在她清醒时归来，三人围坐，唱起生日歌，祝她平安、快乐、健康地长大。

莫宁每次都安静地接受。尽管她知道，母亲以为她无法理解语义与情绪，总在托起她的腿时露出悲伤的神情；也清楚地记得，他们在说到“健康”二字时，那片刻不易察觉的迟疑。但家人对她很好，莫宁很早就理解了这一点。

只有一次是例外。夜半的争吵声甚至将她从睡梦中惊醒，父亲拔高了声音，母亲声泪俱下，陌生的威胁感让她忍不住放声哭泣。她确信父母听到了，因为争吵声中确实出现了一个因她而生的停顿，但紧随其后的，是更高亢的呼喊与更剧烈的歇斯底里。

于是，一种前所未有的责任，在那停顿的间隙里悄然滋生。她必须亲自去阻止他们。

她以床头的轮椅为目标，双手攥紧床榻外侧的护栏，开始挪动身体。先是胸口——这很容易，她探出身，拉近与目标的距离。再是小腹，柔软的脏器被坚硬的木头压向脊柱，带来明确的钝痛，却类似一种成功的预示。最后是双腿，她集中了全部力气，让那不听使唤的肢体也翻越过去——

然后，嘭。

争吵戛然而止。

世界重新被寂静填满。轮椅因她坠落的撞击而侧翻在地。莫宁用手臂撑起痛到发麻的半身，感到额角有一股温热的暖意正缓缓滑过她常年冰凉的皮肤。母亲俯身将她抱入怀中，疲惫地流泪。`,
  },
  {
    title: "沉默",
    desc: `莫宁同学总是很沉默，她故乡的同学大多这么认为。

由于那架笨重的轮椅，她自幼便被隔绝于同龄人的追逐打闹之外。作为替代，她将远超常人的精力全然倾注于课业，用无止境的阅读与演算填满所有时间的缝隙。老师曾数次将她树为典范，直到他们发现自己也渐渐难以跟上莫宁思维的步伐。学生间则流传着私语，说她的父亲服务于新联邦某家声名狼藉的企业，因为每逢放学，总有一位管家模样的人，带着无可指摘却冰冷的神情为她推行轮椅。莫宁静坐其上，如同一个被妥善安置的人偶，只是沉默地顺应着这一切。

更令孩子们隐隐不快的，是她对待善意的消极。即便老师已在全班面前叮嘱，要多多关照行动不便的莫宁；即便同学们常带着慷慨的笑意，询问她是否需要帮助，她也总是垂下头，用垂落的碎发遮蔽视线，再轻轻摇头，以沉默回绝一切。

孩子们的耐心向来有限，当失去作为施赠者的优越，排斥便悄然滋长。不久后，当她推着轮椅独自在走廊缓慢前行，不再有人围拢上去。他们带着心照不宣的沉默彼此相望，为那架笨重的轮椅让出一条笔直的通道，像被分开水流的鱼群。

莫宁其实从未介怀过那些向她递出又收回的关怀，正如她执着于优异的成绩，是因为这是她唯一能向这个世界表达的、无声的谢意。她一直明白，只要她开口求助，无论出于真心还是规训，人们总会向她伸出援手。

但她始终记得母亲那个疲惫的眼神，所以她总是沉默。`,
  },
  {
    title: "录取通知书",
    desc: `16岁时，她收到了星炬学院的入学邀请。

觉醒共鸣能力是在更早些的时候。似乎只是在一个平常的午后，她意识到自己不需要再去用手推动轮椅那沉重的轮毂。这并不是一件格外值得兴奋的事，那些机械的原理她早已了如指掌，让它们在想象中运作，和在现实中操控它们，对莫宁而言并无区别。毕竟，无法行动的自己依然被困于笨重的金属之上。

她在那天夜晚将自己搬向床榻，看见了大腿上那道微微亮起的声痕，像一种精心设计的黑色幽默。

但那张录取通知书不同。

众所周知，星炬学院是全索拉里斯顶尖的人才培养机构，而它所属的深空联合更代表着人类科技的最前沿。那张印刷精美的纸片像是为莫宁逼仄的世界划了开了一道口子——即便无法行动如常，即便拥有的是自己都认为无用的能力，也依然被赋予了“价值”。

即便是这样的自己，也能跻身于照亮人类的星群之列吗？

她很快完成了预科的课程，共鸣能力被认为极适合隧者工学部。她成为了同学口中的天才，导师备受青睐的学生，学部最年轻的助教。但这远远不够。她彻夜钻研课题报告，主动投身有危险性的实验，令自己屡次过度使用共鸣能力，濒临超频的边缘。在一个因为没能按时喝下营养剂而意识恍惚的瞬间里，她想——倘若——只是倘若，这具躯壳最终能成为构筑科学殿堂的基石之一，也算一个完满的结局。

“已经很晚了，不休息么？”

她抬起头，看到一双鎏金色的眼睛。

她记得这个人。她所在项目的发起者，她在星炬学院的前辈。她一直很喜欢这双金色的眼睛。那其中带着某种此时的莫宁尚未知晓的、显得宏大而寂寥的情绪，因此看向任何人时都带着一视同仁的平静，不包含那种她早已习惯的同情。

“不休息的话，”那位前辈发出邀请，声调依旧平稳，“会错过星星的。”`,
  },
  {
    title: "群星",
    desc: `星炬学院没有星星。彼时的大气拟造阵列还只具备最基础的降水功能，还在探索阶段的深空联合也尚无余力对这片陌生的地下世界进行装点。

但莫宁没有想到，那位前辈所说的星星，是冰原之上的世界。

那是一次前往冰原之上的科考。平日没有人会邀请莫宁参加这样的行动，因此在【TA】向自己伸出手时，她反而没有拒绝。

于是她坐上了通往冰原之上的联运椎骨。意外的，大多数时间她都和其他科考队员没什么不同，安坐在交通工具中，一路计算着那些轰鸣的庞大机械收集得来的数据。偶尔，当他们的交通工具无法通行——或者只是单纯的有什么值得一看的景色时，那位前辈会背着她，走在风雪之中。

雪原中的风夹杂着冰片擦过滚烫的面颊。远方的面貌在雪风中模糊不清，但似乎有崭新的世界在她面前豁然展开。

考察的最后，他们抵达了一片机械的残骸，坐在废墟之上，仰望漫天星辰。

“你喜欢星星吗？”

那个人这样说着。莫宁跟随【TA】抬起头，望向那些自光年外而来的碎片。

“正因为能看到星星，人类才会对天空之外的世界产生好奇。”

“我想，你们会证明，这看似永恒的隔绝并非终点。人们终有一日能以自己的意志，迈向星辰大海。”

——啊，原来如此。

她首先意识到的，是他人与自己是同等渺小的存在。她，父亲，母亲，故乡的同学，还有【TA】。在浩瀚无垠的群星下，都只是这颗星球里一颗微不足道的尘埃。

紧接着，她回想起那些被自己刻意压制的痛苦。她想起第一次翻越护栏时坠地的疼痛，想起母亲离开家时被她咽下的哭喊，想起故乡的学校里孩子们秘而不宣的眼神，被让出的过道，想起她想象自己结束一切时那令人不安的平静。

最后，不甘翻涌而来。

那是一种前所未有的情绪，它是如此强烈地席卷了她，以至于像是对某种伟大存在的宣战。一簇冰冷的星火在她沉寂的胸腔点燃，带着灼痛的引力，将她因守于狭小世界而维持的平静撕毁殆尽。

然后，星光倾斜而来。

带着泪水，带着心脏的轰鸣，她像身边人一般伸出手，伸向那无穷远处的星光。

——“我也想……抵达那样的未来。”

`,
  },
  {
    title: "然后，再一步",
    desc: `“莫宁教授，科蒂和维恩斯因为‘赫利俄斯’的测试部件吵起来了。恐怕，还是需要您……”

莫宁叹了口气。此时，她头顶上方的改良星栈正在稳定运行，释放她的共鸣能力操控键盘，回复有关换日仪式进度的邮件；她左手操控着悬浮屏，模拟点火测试中的能量输出参数，右手则进行着一则有关弹塑性转换的计算。即使如此，听到有关人际关系的协调，她还是不得不停下所有的进度，努力不被察觉地发出一声小小哀鸣。

吵架的是两名刚从星炬学院毕业的学生，朝气蓬勃，亟待认可。莫宁没有经历过他们那样的时期，但她想起了自己刚加入研究院时的情景。她为了拟造星空的事第一次和理事会起了冲突，获许建造后，却又有虚质磁暴在那里发生。为了确保工程完成，她超频维持信号塔的运行，又被从天而降的部件砸中腿部……

好吧，似乎的确与此时的孩子们没有那么不同。

当事件处理完毕的时候，大气拟造阵列已经切换到了夜晚的状态。浮游炮为她递来用作晚餐的营养补充剂，是农学部新研发的烟熏牛排味。她拧开盖子，熄灭面前最后一个悬浮屏，看向窗外的星空。

那位前辈已经离开很久了。

那之后发生了很多事。比如毕业那年，她顺利收到了研究院的入职邀请；比如她接手的第一个项目，就是在拉海洛建造一片星空；比如她的腿经过了手术，现在已经可以自如地行走了；比如最后一次见面时，那个人说，拉海洛已经可以靠自己的力量前进了。

那个人不属于拉海洛。那时莫宁才明白，那双鎏金色眼睛中的一视同仁从何而来。【TA】注定不会停留于某处。

但是，没关系。

研究院的试验大厅中，“赫利俄斯”反射出银色的光芒。“换日仪式”计划——那是连那个人也未曾想到的，实现“启动隧者”这一宏大目标的第一步。

——我已经找到我要做的事了，前辈。甚至已经能比你走得更远了。

——所以，当你回来的时候……我是否也能，站在你身边了呢？`,
  },
];

// 时间轴事件（整合官方剧情）
const timelineEvents = [
  {
    title: "缄默的观测者",
    desc: `普攻·广域观测模式免疫打断。
莫宁为目标附加的干涉标记的持续时间延长150%，且干涉标记的伤害提升效果不再需要目标处于【震谐·干涉】或【集谐·干涉】状态。莫宁为目标附加观测标记时，将同时附加干涉标记。`,
  },
  {
    title: "熵增的启明星",
    desc: `附近队伍中所有角色对拥有干涉标记的目标造成的暴击伤害提升：莫宁自身共鸣效率大于100%时，每超过1%的共鸣效率计算为0.2%的暴击伤害提升，上限32%。
谐振场、强谐振场还会使生效范围内附近队伍中所有角色偏谐值累积效率额外提升20%。`,
  },
  {
    title: "递归的蓝图",
    desc: `施放共鸣技能·分布式阵列时，额外回复自身25点协奏能量，并回复100点【相对动能】，每25秒可触发1次。`,
  },
  { title: "宇宙的隐变量", desc: "强谐振场治疗量提升30%" },
  {
    title: "钟慢效应",
    desc: `共鸣解放·临界协议伤害倍率提升40%。
震谐响应·粒子射流伤害倍率提升160%。`,
  },
  {
    title: "直到群星的尽头",
    desc: `共鸣解放·临界协议造成的伤害提升400%。
莫宁处于非战斗状态大于4秒后，每0.2秒回复共鸣能量上限10%的共鸣能量。`,
  },
];

// 共鸣链数据
const chains = [
  {
    title: "自拍照",
    desc: "天气调到雨天，莫宁会在飞讯发一张自拍照",
  },
  { title: "常驻点位", desc: "晚上11点，天气流星雨在赫尔琉斯悬崖边(在睡觉)" },
  { title: "共乘", desc: "莫宁共乘的时候会抱住漂子，其他角色是侧坐" },
];

const current = ref(0);
const reducedMotion = window.matchMedia?.(
  "(prefers-reduced-motion: reduce)"
).matches;
const storyRef = ref<HTMLElement | null>(null);

function prevStory() {
  current.value = (current.value - 1 + stories.length) % stories.length;
  storyRef.value?.scrollIntoView({ behavior: "smooth", block: "start" });
}
function nextStory() {
  current.value = (current.value + 1) % stories.length;
  storyRef.value?.scrollIntoView({ behavior: "smooth", block: "start" });
}

// ========== 浮动小人逻辑 ==========
interface Chibi {
  src: string;
  top: number;
  left: number;
}
const chibiList = ref<Chibi[]>([]);

onMounted(async () => {
  if (!reducedMotion) {
    // 入场动画
    const tl = gsap.timeline({ defaults: { ease: "power3.out" } });
    tl.from(".avatar-card", { duration: 0.8, y: 30, opacity: 0, scale: 0.95 })
      .from(
        ".right-col > .hero-panel",
        { duration: 0.8, x: 30, opacity: 0 },
        "-=0.4"
      )
      .from(
        ".hero-stats .stat-item",
        { duration: 0.5, scale: 0, opacity: 0, stagger: 0.1 },
        "-=0.2"
      );

    // 卡片滚动动画
    gsap.utils.toArray(".card").forEach((card: any, idx: number) => {
      gsap.from(card, {
        scrollTrigger: {
          trigger: card,
          start: "top 85%",
          toggleActions: "play none none reverse",
        },
        y: 40,
        opacity: 0,
        duration: 0.8,
        delay: idx * 0.1,
      });
    });

    // 时间轴条目动画
    gsap.utils.toArray(".timeline-item").forEach((item: any, idx: number) => {
      gsap.from(item, {
        scrollTrigger: {
          trigger: item,
          start: "top 85%",
          toggleActions: "play none none reverse",
        },
        x: idx % 2 === 0 ? -30 : 30,
        opacity: 0,
        duration: 0.6,
        delay: idx * 0.1,
      });
    });

    // 头像呼吸
    gsap.to(".avatar-card", {
      scale: 1.02,
      duration: 2.5,
      yoyo: true,
      repeat: -1,
      ease: "sine.inOut",
    });

    // 透明腿特效
    gsap.to(".leg-glow", {
      opacity: 0.8,
      duration: 1.5,
      yoyo: true,
      repeat: -1,
      ease: "sine.inOut",
    });
  }

  // 初始化浮动小人
  const total = 10;
  let pickCount = 3;
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const isMobile = window.innerWidth <= 768;
  const imgWidth = 100;
  const imgHeight = 100;

  function shuffle(array: any[]) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  if (isMobile) pickCount = 1;
  const nums = shuffle(Array.from({ length: total }, (_, k) => k + 1));
  const picks = nums.slice(0, pickCount);

  chibiList.value = picks.map((i) => ({
    src: `/QImages/1 (${i}).png`,
    left: Math.random() * (vw - imgWidth),
    top: Math.random() * (vh - imgHeight),
  }));

  await nextTick();

  const imgs = document.querySelectorAll<HTMLImageElement>(".chibi-img");
  const padding = 200;

  imgs.forEach((img, index) => {
    gsap.fromTo(
      img,
      { opacity: 0, scale: 0.5 },
      {
        opacity: 1,
        scale: 1,
        duration: 0.8,
        ease: "back.out(2)",
        delay: 0.2 * index,
      }
    );

    const animate = (img: HTMLImageElement) => {
      let { x, y } = img.getBoundingClientRect();
      let deltaX = (Math.random() - 0.5) * 200;
      let deltaY = (Math.random() - 0.5) * 200;

      let nextX = x + deltaX;
      let nextY = y + deltaY;

      if (nextX < padding) deltaX = padding - x;
      if (nextX + img.width > window.innerWidth - padding)
        deltaX = window.innerWidth - padding - (x + img.width);
      if (nextY < padding) deltaY = padding - y;
      if (nextY + img.height > window.innerHeight - padding)
        deltaY = window.innerHeight - padding - (y + img.height);

      gsap.to(img, {
        x: `+=${deltaX.toFixed(0)}`,
        y: `+=${deltaY.toFixed(0)}`,
        rotation: `+=${((Math.random() - 0.5) * 60).toFixed(0)}`,
        duration: 2 + Math.random() * 2,
        ease: "power1.inOut",
        onComplete: () => animate(img),
      });
    };

    img.addEventListener("mouseenter", () => {
      gsap.killTweensOf(img);
      gsap.to(img, {
        x: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        y: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        duration: 1.2,
        ease: "back.out(2)",
        onComplete: () => animate(img),
      });
    });

    animate(img);
  });
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

/* 莫宁配色 */
$bg-deep: #0a0c16;
$bg-panel: #121826;
$accent-blue: #4aa5ff;
$accent-light: #9ac9ff;
$red-pupil: #ff6b8b;
$text-primary: #f0f6ff;
$text-muted: rgba(240, 246, 255, 0.6);

.morning-page {
  position: relative;
  min-height: 100vh;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, $bg-deep);
  color: $text-primary;
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
  padding: clamp(16px, 4vw, 40px);
  padding-top: 80px;
  overflow-x: hidden;

  &.reduced-motion * {
    transition: none !important;
    animation: none !important;
  }

  /* 静态机械纹理 */
  .grid-bg,
  .hex-bg {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 0;
  }
  .grid-bg {
    background-image: linear-gradient(
        rgba(74, 165, 255, 0.02) 1px,
        transparent 1px
      ),
      linear-gradient(90deg, rgba(74, 165, 255, 0.02) 1px, transparent 1px);
    background-size: 40px 40px;
  }
  .hex-bg {
    background-image: repeating-linear-gradient(
      45deg,
      rgba(74, 165, 255, 0.02) 0px,
      rgba(74, 165, 255, 0.02) 2px,
      transparent 2px,
      transparent 30px
    );
  }

  /* 动态光晕层 */
  .glow-orb {
    position: absolute;
    border-radius: 50%;
    filter: blur(60px);
    z-index: 0;
    opacity: 0.3;
    animation: glowPulse 8s infinite alternate;
  }
  .glow-1 {
    top: 10%;
    left: 5%;
    width: 400px;
    height: 400px;
    background: radial-gradient(circle, $accent-blue, transparent 70%);
  }
  .glow-2 {
    bottom: 20%;
    right: 10%;
    width: 500px;
    height: 500px;
    background: radial-gradient(circle, $accent-light, transparent 70%);
    animation-delay: 2s;
  }
  .glow-3 {
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 800px;
    height: 800px;
    background: radial-gradient(
      circle,
      rgba(74, 165, 255, 0.2),
      transparent 80%
    );
    animation: glowPulse 12s infinite alternate;
  }

  /* 扫描线 & 数据流 */
  .scan-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 2px;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(74, 165, 255, 0.5) 20%,
      rgba(200, 230, 255, 0.7) 50%,
      rgba(74, 165, 255, 0.5) 80%,
      transparent
    );
    filter: blur(3px);
    animation: scanMove 6s linear infinite;
    z-index: 2;
    pointer-events: none;
    opacity: 0.5;
  }
  .data-flow {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(
      90deg,
      transparent,
      $accent-blue 20%,
      $accent-light 50%,
      $accent-blue 80%,
      transparent
    );
    filter: blur(2px);
    animation: flowMove 4s linear infinite;
    z-index: 2;
    opacity: 0.6;
  }

  /* 红瞳微光 */
  .red-dot {
    position: absolute;
    width: 4px;
    height: 4px;
    background: $red-pupil;
    border-radius: 50%;
    filter: blur(2px);
    box-shadow: 0 0 12px $red-pupil;
    opacity: 0.5;
    animation: redBreath 3s infinite;
    z-index: 2;
    pointer-events: none;
  }
  .red-dot-1 {
    top: 15%;
    left: 10%;
  }
  .red-dot-2 {
    top: 25%;
    right: 15%;
  }
  .red-dot-3 {
    bottom: 20%;
    left: 20%;
  }
  .red-dot-4 {
    bottom: 30%;
    right: 25%;
  }

  /* 机械角 */
  .mech-corner {
    position: absolute;
    top: 20px;
    left: 20px;
    width: 60px;
    height: 60px;
    background: radial-gradient(
      circle at 30% 30%,
      rgba(74, 165, 255, 0.15),
      transparent 70%
    );
    clip-path: polygon(
      25% 0%,
      75% 0%,
      100% 25%,
      100% 75%,
      75% 100%,
      25% 100%,
      0% 75%,
      0% 25%
    );
    animation: rotateSlow 20s linear infinite;
    z-index: 2;
    opacity: 0.3;
  }

  /* 旋转数据环 */
  .data-ring {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 800px;
    height: 800px;
    border: 1px solid rgba(74, 165, 255, 0.1);
    border-radius: 50%;
    box-shadow: 0 0 50px rgba(74, 165, 255, 0.1);
    animation: ringRotate 40s linear infinite;
    z-index: 1;
    pointer-events: none;
    &::after {
      content: "";
      position: absolute;
      inset: -20px;
      border: 1px dashed rgba(74, 165, 255, 0.05);
      border-radius: 50%;
      animation: ringRotateReverse 30s linear infinite;
    }
  }

  /* 闪烁粒子 */
  .floating-particles {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 1;
  }
  .particle {
    position: absolute;
    width: 2px;
    height: 2px;
    background: $accent-light;
    border-radius: 50%;
    box-shadow: 0 0 8px $accent-blue;
    opacity: 0.3;
    animation: floatParticle 15s infinite;
    &:nth-child(1) {
      top: 10%;
      left: 20%;
      animation-delay: 0s;
    }
    &:nth-child(2) {
      top: 30%;
      left: 70%;
      animation-delay: 1s;
    }
    &:nth-child(3) {
      top: 50%;
      left: 40%;
      animation-delay: 2s;
    }
    &:nth-child(4) {
      top: 70%;
      left: 80%;
      animation-delay: 3s;
    }
    &:nth-child(5) {
      top: 20%;
      left: 90%;
      animation-delay: 4s;
    }
    &:nth-child(6) {
      top: 80%;
      left: 10%;
      animation-delay: 5s;
    }
    &:nth-child(7) {
      top: 40%;
      left: 30%;
      animation-delay: 6s;
    }
    &:nth-child(8) {
      top: 60%;
      left: 60%;
      animation-delay: 7s;
    }
    &:nth-child(9) {
      top: 15%;
      left: 50%;
      animation-delay: 8s;
    }
    &:nth-child(10) {
      top: 85%;
      left: 40%;
      animation-delay: 9s;
    }
    &:nth-child(11) {
      top: 25%;
      left: 10%;
      animation-delay: 10s;
    }
    &:nth-child(12) {
      top: 75%;
      left: 85%;
      animation-delay: 11s;
    }
  }

  /* 透明腿特效 */
  .leg-glow {
    position: fixed;
    bottom: 50px;
    right: 50px;
    width: 300px;
    height: 300px;
    background: radial-gradient(
      circle at 30% 50%,
      rgba(74, 165, 255, 0.2),
      transparent 70%
    );
    filter: blur(30px);
    z-index: 5;
    pointer-events: none;
    animation: legPulse 3s infinite alternate;
  }

  /* 动画定义 */
  @keyframes glowPulse {
    0% {
      opacity: 0.2;
      transform: scale(1);
    }
    100% {
      opacity: 0.5;
      transform: scale(1.2);
    }
  }
  @keyframes scanMove {
    0% {
      top: 0;
      opacity: 0.3;
    }
    10% {
      opacity: 0.8;
    }
    90% {
      opacity: 0.8;
    }
    100% {
      top: 100%;
      opacity: 0.3;
    }
  }
  @keyframes flowMove {
    0% {
      transform: translateX(-100%);
    }
    100% {
      transform: translateX(100%);
    }
  }
  @keyframes redBreath {
    0%,
    100% {
      opacity: 0.2;
      transform: scale(1);
    }
    50% {
      opacity: 0.8;
      transform: scale(1.5);
    }
  }
  @keyframes rotateSlow {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
  @keyframes ringRotate {
    from {
      transform: translate(-50%, -50%) rotate(0deg);
    }
    to {
      transform: translate(-50%, -50%) rotate(360deg);
    }
  }
  @keyframes ringRotateReverse {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(-360deg);
    }
  }
  @keyframes floatParticle {
    0% {
      transform: translate(0, 0);
      opacity: 0.2;
    }
    25% {
      transform: translate(30px, -30px);
      opacity: 0.6;
    }
    50% {
      transform: translate(-20px, 40px);
      opacity: 0.3;
    }
    75% {
      transform: translate(20px, 20px);
      opacity: 0.5;
    }
    100% {
      transform: translate(0, 0);
      opacity: 0.2;
    }
  }
  @keyframes legPulse {
    0% {
      opacity: 0.3;
      transform: scale(1);
    }
    100% {
      opacity: 0.7;
      transform: scale(1.3);
    }
  }

  .container {
    position: relative;
    z-index: 10;
    max-width: 1200px;
    margin: 0 auto;
  }

  /* HERO - 增强版 */
  .hero {
    display: grid;
    grid-template-columns: 280px 1fr;
    gap: 24px;
    margin-bottom: 24px;

    .left-col {
      .avatar-card {
        position: relative;
        width: 100%;
        background: rgba(10, 15, 30, 0.5);
        backdrop-filter: blur(12px);
        border: 1px solid rgba(74, 165, 255, 0.2);
        border-radius: 24px;
        padding: 16px;
        transition: transform 0.3s, box-shadow 0.3s;
        overflow: hidden;

        .avatar-glow {
          position: absolute;
          inset: 0;
          background: radial-gradient(
            circle at 30% 30%,
            $accent-blue,
            transparent 70%
          );
          opacity: 0;
          transition: opacity 0.3s;
          filter: blur(30px);
        }

        &:hover {
          transform: translateY(-4px) scale(1.02);
          box-shadow: 0 30px 60px rgba(0, 0, 0, 0.7), 0 0 50px $accent-blue;
          .avatar-glow {
            opacity: 0.3;
          }
        }

        .avatar-wrap {
          width: 100%;
          aspect-ratio: 1;
          overflow: hidden;
          border-radius: 16px;
          margin-bottom: 12px;
        }
        .avatar {
          width: 100%;
          height: 100%;
          object-fit: cover;
          transition: transform 0.3s;
        }
        &:hover .avatar {
          transform: scale(1.05);
        }
        .avatar-meta {
          display: flex;
          flex-direction: column;
          gap: 8px;
        }
        .avatar-badge {
          background: rgba(74, 165, 255, 0.15);
          border: 1px solid rgba(74, 165, 255, 0.3);
          border-radius: 30px;
          padding: 4px 12px;
          font-size: 0.9rem;
          color: $accent-light;
          text-align: center;
        }
      }
    }

    .right-col {
      .hero-panel {
        background: rgba(10, 15, 30, 0.5);
        backdrop-filter: blur(12px);
        border: 1px solid rgba(74, 165, 255, 0.2);
        border-radius: 24px;
        padding: 24px;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        position: relative;
        overflow: hidden;

        &::before {
          content: "";
          position: absolute;
          inset: 0;
          background: radial-gradient(
            circle at 70% 30%,
            $accent-blue,
            transparent 70%
          );
          opacity: 0.1;
          filter: blur(40px);
        }

        .name {
          margin: 0 0 8px;
          font-family: "Orbitron", sans-serif;
          font-size: clamp(2rem, 4vw, 2.8rem);
          background: linear-gradient(135deg, #fff, $accent-light);
          background-clip: text;
          -webkit-background-clip: text;
          color: transparent;
          position: relative;
          &::after {
            content: "";
            position: absolute;
            left: 0;
            bottom: -4px;
            width: 100px;
            height: 2px;
            background: linear-gradient(90deg, $accent-blue, transparent);
            animation: titleLine 3s infinite;
          }
          .jp {
            font-size: 1rem;
            color: $text-muted;
            margin-left: 12px;
          }
        }
        @keyframes titleLine {
          0% {
            width: 0;
            opacity: 0;
          }
          50% {
            width: 100px;
            opacity: 1;
          }
          100% {
            width: 0;
            opacity: 0;
          }
        }
        .subtitle {
          font-size: 1.1rem;
          color: $accent-blue;
          margin-bottom: 12px;
        }
        .lead {
          line-height: 1.6;
          color: $text-primary;
          opacity: 0.9;
          margin-bottom: 20px;
        }
        .hero-stats {
          display: flex;
          gap: 16px;
          flex-wrap: wrap;
          .stat-item {
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(74, 165, 255, 0.2);
            border-radius: 30px;
            padding: 4px 16px;
            font-size: 0.9rem;
            transition: all 0.2s;
            &:hover {
              background: rgba(74, 165, 255, 0.15);
              border-color: $accent-blue;
            }
            span {
              color: $accent-light;
              margin-right: 6px;
            }
          }
        }
      }
    }
  }

  /* 通用卡片 */
  .card {
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(74, 165, 255, 0.2);
    border-radius: 24px;
    padding: 24px;
    margin-bottom: 24px;
    transition: transform 0.3s, box-shadow 0.3s;

    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6),
        0 0 30px rgba(74, 165, 255, 0.2);
    }

    .card-head {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;

      h2 {
        margin: 0;
        font-family: "Orbitron", sans-serif;
        font-size: 1.5rem;
        color: $accent-light;
        position: relative;
        display: inline-block;
        &::before {
          content: "【";
          margin-right: 8px;
          color: $accent-blue;
          opacity: 0.5;
        }
        &::after {
          content: "】";
          margin-left: 8px;
          color: $accent-blue;
          opacity: 0.5;
        }
      }
    }
  }
  /* 双栏布局 */
  .two-col.reports-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 24px;

    .report-card {
      background: rgba(10, 15, 30, 0.5);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(74, 165, 255, 0.2);
      border-radius: 24px;
      padding: 20px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
      transition: transform 0.3s, box-shadow 0.3s;
      position: relative;
      overflow: hidden;

      &::before {
        content: "";
        position: absolute;
        inset: 0;
        background: radial-gradient(
          circle at 20% 30%,
          rgba(74, 165, 255, 0.1),
          transparent 70%
        );
        pointer-events: none;
      }

      &:hover {
        transform: translateY(-5px);
        box-shadow: 0 30px 50px rgba(0, 0, 0, 0.6),
          0 0 30px rgba(74, 165, 255, 0.2);
      }

      .report-header {
        display: flex;
        align-items: center;
        gap: 10px;
        margin-bottom: 16px;
        border-bottom: 1px solid rgba(74, 165, 255, 0.2);
        padding-bottom: 12px;

        .report-icon {
          font-size: 1.8rem;
          filter: drop-shadow(0 0 8px $accent-blue);
        }

        .section-subtitle {
          margin: 0;
          font-size: 1.2rem;
          color: $accent-blue;
          font-weight: 600;
          font-family: "Orbitron", sans-serif;
        }
      }

      .report-content {
        color: rgba(255, 255, 255, 0.9);
        font-size: 0.95rem;
        line-height: 1.6;

        .report-meta {
          display: flex;
          gap: 8px;
          background: rgba(0, 0, 0, 0.2);
          padding: 6px 12px;
          border-radius: 30px;
          margin-bottom: 16px;
          font-family: "Share Tech Mono", monospace;
          border: 1px solid rgba(74, 165, 255, 0.2);

          .label {
            color: $accent-light;
          }
        }

        .report-section {
          margin-bottom: 20px;

          h4 {
            color: $accent-light;
            font-size: 1rem;
            margin-bottom: 8px;
            font-weight: 600;
            display: flex;
            align-items: center;
            gap: 6px;

            &::before {
              content: "▌";
              color: $accent-blue;
            }
          }

          p {
            margin-bottom: 8px;
          }

          .badge-group {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            margin-bottom: 12px;
          }

          .badge {
            display: inline-block;
            background: rgba(74, 165, 255, 0.15);
            border: 1px solid rgba(74, 165, 255, 0.3);
            border-radius: 30px;
            padding: 2px 10px;
            font-size: 0.85rem;
            color: $accent-light;
            margin-right: 8px;
            white-space: nowrap;

            &.success {
              background: rgba(0, 200, 100, 0.15);
              border-color: rgba(0, 200, 100, 0.3);
              color: #a0ffb0;
            }
            &.warn {
              background: rgba(255, 200, 0, 0.15);
              border-color: rgba(255, 200, 0, 0.3);
              color: #ffe070;
            }
          }

          .spec-item {
            display: flex;
            margin-bottom: 6px;

            .spec-label {
              min-width: 100px;
              color: $accent-light;
            }
            .spec-value {
              color: #fff;
            }
          }

          .note {
            background: rgba(74, 165, 255, 0.1);
            border-left: 4px solid $accent-blue;
            padding: 10px 12px;
            border-radius: 0 12px 12px 0;
            margin-top: 12px;
            font-style: italic;
            color: rgba(255, 255, 255, 0.8);

            &::before {
              content: "💬 ";
              color: $accent-blue;
            }
          }

          .history-note {
            background: rgba(255, 200, 0, 0.05);
            border-left: 4px solid #ffaa00;
            padding: 8px 12px;
            border-radius: 0 8px 8px 0;
            margin-top: 8px;
          }
        }

        .report-footer {
          margin-top: 20px;
          border-top: 1px dashed rgba(74, 165, 255, 0.3);
          padding-top: 16px;

          .quote {
            font-style: italic;
            color: $accent-light;
            text-align: center;
            font-size: 0.9rem;
          }
        }
      }
    }

    // 响应式移动端
    @media (max-width: 768px) {
      grid-template-columns: 1fr;
      gap: 16px;

      .report-card {
        padding: 16px;

        .report-header {
          .section-subtitle {
            font-size: 1.1rem;
          }
          .report-icon {
            font-size: 1.6rem;
          }
        }

        .report-content {
          font-size: 0.9rem;

          .report-meta {
            flex-wrap: wrap;
            gap: 4px;
            font-size: 0.85rem;
          }

          .badge {
            white-space: normal; // 允许徽章内文字换行
            word-break: break-word;
          }

          .spec-item {
            flex-direction: column;
            align-items: flex-start;
            margin-bottom: 12px;

            .spec-label {
              min-width: auto;
              margin-bottom: 2px;
            }
          }

          .note {
            font-size: 0.85rem;
          }

          .history-note {
            font-size: 0.85rem;
          }

          .quote {
            font-size: 0.85rem;
          }
        }
      }
    }

    // 极小屏幕额外调整
    @media (max-width: 480px) {
      .report-card {
        padding: 12px;

        .report-header {
          gap: 6px;
          .section-subtitle {
            font-size: 1rem;
          }
        }

        .badge-group {
          flex-direction: column;
          align-items: flex-start;
        }
      }
    }
  }

  /* 战斗解析 */
  .personality .traits {
    list-style: none;
    padding: 0;
    margin: 0;

    li {
      padding: 12px 0;
      border-bottom: 1px solid rgba(74, 165, 255, 0.1);
      font-size: 1rem;

      .trait-highlight {
        color: $accent-light;
        font-weight: 600;
        margin-right: 8px;
      }
    }
  }
  /* 物品卡片 */
  .items-section .items-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;

    .item-card {
      background: rgba(0, 0, 0, 0.2);
      border: 1px solid rgba(74, 165, 255, 0.15);
      border-radius: 16px;
      padding: 20px;
      transition: all 0.4s cubic-bezier(0.2, 0.9, 0.3, 1);
      position: relative;
      overflow: hidden;
      p {
        white-space: pre-wrap;
      }
      &::before {
        content: "";
        position: absolute;
        inset: 0;
        background: radial-gradient(
          circle at 50% 0%,
          $accent-blue,
          transparent 70%
        );
        opacity: 0;
        transition: opacity 0.3s;
        filter: blur(20px);
      }

      &:hover {
        transform: translateY(-10px) scale(1.05);
        border-color: $accent-blue;
        box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 30px $accent-blue;
        &::before {
          opacity: 0.2;
        }
        .item-image img {
          transform: scale(1.1);
        }
      }

      @keyframes iconFloat {
        0% {
          transform: translateY(0);
        }
        100% {
          transform: translateY(-5px);
        }
      }
      .item-image {
        width: 100%;
        margin-bottom: 12px;
        img {
          width: 100%;
          border-radius: 12px;
          transition: transform 0.3s;
        }
      }
      h3 {
        font-size: 1.1rem;
        color: $accent-light;
        margin-bottom: 8px;
      }
      p {
        font-size: 0.9rem;
        line-height: 1.5;
        color: rgba(255, 255, 255, 0.8);
      }
    }
  }

  /* 故事模块 */
  .stories-section {
    .card {
      overflow: hidden;
    }
    .story-content {
      min-height: 180px;
      padding: 20px;
      background: rgba(0, 0, 0, 0.1);
      border-radius: 16px;
      .story-item {
        h3 {
          color: $accent-blue;
          margin-bottom: 12px;
          font-size: 1.3rem;
        }
        p {
          line-height: 1.8;
          white-space: pre-wrap;
        }
      }
    }
    .story-nav {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: 20px;
      .nav-btn {
        background: transparent;
        border: 1px solid rgba(74, 165, 255, 0.3);
        border-radius: 30px;
        color: $accent-light;
        padding: 8px 24px;
        cursor: pointer;
        transition: all 0.2s;
        &:hover {
          background: rgba(74, 165, 255, 0.1);
          border-color: $accent-blue;
          transform: scale(1.05);
        }
      }
      .pager {
        color: $text-muted;
      }
    }
  }

  /* 时间轴 */
  .timeline-section {
    position: relative;
    overflow: visible;

    .timeline-container {
      position: relative;
      padding: 20px 0;
    }

    .timeline-line {
      position: absolute;
      left: 50%;
      top: 0;
      bottom: 0;
      width: 2px;
      background: linear-gradient(
        180deg,
        $accent-blue,
        $accent-light,
        transparent
      );
      transform: translateX(-50%);
      animation: timelineGlow 3s infinite;

      @keyframes timelineGlow {
        0% {
          opacity: 0.3;
          box-shadow: 0 0 5px $accent-blue;
        }
        50% {
          opacity: 1;
          box-shadow: 0 0 20px $accent-blue;
        }
        100% {
          opacity: 0.3;
          box-shadow: 0 0 5px $accent-blue;
        }
      }
    }

    .timeline-item {
      position: relative;
      width: 50%;
      padding: 0 30px;
      margin-bottom: 40px;
      opacity: 0;
      animation: timelineAppear 0.8s forwards;

      &.timeline-right {
        margin-left: 50%;
        .timeline-marker {
          left: -25px;
        }
      }

      .timeline-marker {
        position: absolute;
        top: 0;
        right: -25px;
        width: 50px;
        height: 50px;
        display: flex;
        flex-direction: column;
        align-items: center;
        z-index: 2;

        .marker-dot {
          width: 16px;
          height: 16px;
          background: $accent-light;
          border-radius: 50%;
          box-shadow: 0 0 20px $accent-blue;
          animation: dotPulse 2s infinite;
        }
      }

      .timeline-card {
        background: rgba(0, 0, 0, 0.3);
        border: 1px solid rgba(74, 165, 255, 0.2);
        border-radius: 16px;
        padding: 16px;
        transition: all 0.3s;

        &:hover {
          transform: translateY(-5px);
          border-color: $accent-blue;
          box-shadow: 0 10px 30px rgba(74, 165, 255, 0.2);
        }

        h4 {
          color: $accent-light;
          margin-bottom: 8px;
          font-size: 1.1rem;
        }
        p {
          font-size: 0.9rem;
          line-height: 1.5;
          color: rgba(255, 255, 255, 0.8);
          margin: 0;
          white-space: pre-wrap;
        }
      }
    }

    @keyframes timelineAppear {
      to {
        opacity: 1;
        transform: translateY(0);
      }
    }
    .timeline-item:not(.timeline-right) {
      transform: translateX(-20px);
    }
    .timeline-item.timeline-right {
      transform: translateX(20px);
    }
  }

  /* 共鸣链 */
  .chains-section .chains-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 16px;

    .chain-card {
      background: rgba(0, 0, 0, 0.2);
      border: 1px solid rgba(74, 165, 255, 0.1);
      border-radius: 16px;
      padding: 16px;
      transition: all 0.3s;
      position: relative;
      overflow: hidden;
      animation: cardFloat 0.8s forwards;

      &:hover {
        transform: translateY(-5px);
        border-color: $accent-blue;
        box-shadow: 0 0 30px rgba(74, 165, 255, 0.2);
        .chain-number {
          background: $accent-blue;
          color: $bg-deep;
        }
      }

      .chain-number {
        width: 30px;
        height: 30px;
        background: rgba(74, 165, 255, 0.2);
        border: 1px solid $accent-blue;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
        margin-bottom: 12px;
        transition: all 0.3s;
      }
      h4 {
        color: $accent-light;
        margin-bottom: 8px;
        font-size: 1rem;
      }
      p {
        font-size: 0.85rem;
        color: rgba(255, 255, 255, 0.8);
        line-height: 1.5;
      }
    }
  }

  /* 浮动小人 */
  .floating-chibis {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 100;

    .chibi-img {
      position: absolute;
      width: 80px;
      pointer-events: auto;
      z-index: 101;
      transition: filter 0.3s;
      filter: drop-shadow(0 0 10px $accent-blue);
      &:hover {
        filter: drop-shadow(0 0 20px $accent-light);
      }
    }
  }

  /* 动画 */
  @keyframes cardFloat {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .fade-slide-enter-active,
  .fade-slide-leave-active {
    transition: all 0.3s ease;
  }
  .fade-slide-enter-from {
    opacity: 0;
    transform: translateY(20px);
  }
  .fade-slide-leave-to {
    opacity: 0;
    transform: translateY(-20px);
  }

  /* 移动端响应式 */
  @media (max-width: 920px) {
    .hero {
      grid-template-columns: 1fr;
    }
    .personality .traits-grid {
      grid-template-columns: 1fr;
    }
    .items-section .items-grid {
      grid-template-columns: repeat(2, 1fr);
    }
    .chains-section .chains-grid {
      grid-template-columns: repeat(2, 1fr);
    }

    .timeline-section {
      .timeline-line {
        left: 30px;
      }
      .timeline-item {
        width: 100%;
        padding-left: 60px;
        padding-right: 20px;
        &.timeline-right {
          margin-left: 0;
          .timeline-marker {
            left: 10px;
          }
        }
        .timeline-marker {
          left: 10px;
          right: auto;
          flex-direction: row;
          gap: 10px;
          width: auto;
        }
      }
    }
  }
  @media (max-width: 520px) {
    .items-section .items-grid {
      grid-template-columns: 1fr;
    }
    .chains-section .chains-grid {
      grid-template-columns: 1fr;
    }

    .hero-stats {
      flex-direction: column;
    }
    .data-ring {
      display: none;
    }
  }
}
</style>
