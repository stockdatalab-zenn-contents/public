---
title: "2025年7月上旬～10月上旬の技術動向をキーワードマップで俯瞰する"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["自然言語", "NLP", "生成AI", "技術動向"]
published: true
---
## はじめに
こんにちは、マチです。駆け出しデータサイエンティストとして、AIや自然言語処理などの技術トレンドを日々追いかけています。

直近約３か月間のAI関連のニュースや論文紹介サイトを中心に情報収集を行い、得られた記事を要約した上で単語単位に分割し、**出現頻度や共起関係をもとにキーワードマップを作成**しました。まずは、話題になったテーマの「全体像」を見てみましょう。
![](/images/20251014_idea_gathering_info_fords/all.png =700x)
...いやー単語が多すぎて、これだけでは何が中心的な話題なのか分かりづらいですね。ここからは円が大きく目立つ個々のトピックに焦点を当てて、具体的な動向を追ってみます。


## 個別トピックで見るトレンド
### 1.「生成」分野：Sora2の登場が象徴する進化
![](/images/20251014_idea_gathering_info_fords/create.png =700x)
10/1にOpenAIがテキストや画像から動画を生成できるAI「**Sora2**」と、iOSアプリ「Sora」を公開しました。映像に同期した音声の自動生成や、衝突などの物理法則の再現度の高さ、一つのストーリーで複数のシーンを一貫性を保ちながら生成できることなどが、特徴のようです。

これまでデータサイエンスが扱ってきたのは主に**静的データ（テーブルデータ・画像・テキスト）でしたが、今や音楽や映像など動的コンテンツへと拡張**されています。広告・映像制作・メディア運用など、クリエイティブ分野のワークフローにどのような変化をもたらすのか、今後の注目ポイントです。
:::details 参考文献
https://ledge-ai.the-ai.jp/articles/sora2_openai_ios_app_launch
https://techgym.jp/column/sora2/
:::

### 2.「課題・問題」分野：AIの“万能感”に潜む限界
![](/images/20251014_idea_gathering_info_fords/problem.png =700x)
「課題・問題」を中心に見てみると、「忘却」「ハルシネーション」「倫理」「精度」など、AIが抱える課題が浮かび上がります。AIの進化スピードが速い一方で、**信頼性や説明責任**に関する懸念は依然として残っています。

特に興味深かったのは、参考文献の３つの記事です。これらの課題を理解しながら活用していく姿勢が、今後ますます重要になるでしょう。
:::details 参考文献
https://ai-scholar.tech/articles/llm-paper/forget-me-not
https://ledge.ai/articles/openai_apollo_ai_scheming_alignment
https://ledge-ai.the-ai.jp/articles/openai_explains_hallucination_causes
:::


### 3.「プロンプト」分野：人がAIに“伝える力”の重要性
![](/images/20251014_idea_gathering_info_fords/prompt.png =700x)
生成AIはモデルごとに癖があります。同じ指示でも結果が異なるため、**AIの特性に合わせたプロンプト設計**が重要になります。
:::details 参考文献
https://note.com/google_gemini/n/nbe404b055d37
https://pixelgnarly.com/gen4-prompt/
:::
また、興味深い研究として「LLMに的確な指示を出せる人とそうでない人の脳活動の違い」を分析した論文もありました。AIを活用するには、単なる操作スキルだけでなく、**問題の捉え方や発想の順序を変える力**が必要かもしれません。
:::details 参考文献
https://ledge.ai/articles/llm_prompting_brain_fmri_study
https://note.com/miccell/n/ne0dc66a374e5
:::


### 4.「効率」分野：AI活用がもたらす“生産性の再設計”
![](/images/20251014_idea_gathering_info_fords/efficiency.png =700x)
「効率」に関連するキーワードには、**業務・学習・モデルの計算処理や運用の最適化**に関連する単語が並びました。単に「時短ツール」としてではなく、AIを前提とした**新しいワークフロー設計**への移行が進んでいます。

企業がAIを導入する際、ボトルネックとなるのは「個人の習熟度」よりも「システムの組み込み方」だと感じます。「運用効率をどう高めるか」に関する研究・実装が今後はさらに注目されそうです。
:::details 参考文献
https://pythonandai.com/genspark-smart-phone/
http://ai-biblio.com/books/archives/242
http://ai-biblio.com/books/archives/265
https://biggo.jp/news/202503170934_RubyLLM_Elegant_AI_Integration
https://atmarkit.itmedia.co.jp/ait/articles/2510/01/news012.html
https://atmarkit.itmedia.co.jp/ait/articles/2510/10/news013.html
:::


### 5.「自然言語」分野：言葉を扱うAIの“深化と多様化”
![](/images/20251014_idea_gathering_info_fords/nlp.png =700x)
自然言語分野では、翻訳・行政文書解析・教育など、応用範囲の広がりが見られました。
- **官公庁文書に特化した生成AIの開発**
データは古いものの、行政文書処理という新しい応用が期待されます。
- **翻訳特化型LLM「Plamo-2」登場**
海外の一次情報へのアクセスが容易になる一方で、情報の取捨選択の重要性が増していきそうです。
- **児童向けAI書籍の登場**
AI教育が子どもの読書体験に入り始めたことも印象的です。
:::details 参考文献
https://www.itmedia.co.jp/news/articles/2510/02/news102.html
https://www.gizmodo.jp/2025/10/plamo-2-translate.html
http://ai-biblio.com/books/archives/113
http://ai-biblio.com/books/archives/273
:::


## おわりに
この3か月のトレンドを振り返ると、AIは単なるツールを超えて、人間の思考や表現の拡張装置へと進化していることが分かります。一方で、忘却・精度・倫理といった根本的な課題もなお残されています。「どう使うか」だけでなく、「何を任せ、何を自分で考えるか」を意識することが重要になっていきそうです。

