---
title: ステートレス、状態の有効期限、履歴の有効期限
description: 履歴の有効期限とステートレスなイーサリアムの説明
lang: ja
---

# ステートレス、状態の有効期限、履歴の有効期限 {#statelessness}

真に分散化するためには、一般的なハードウェアでイーサリアムノードを実行できることが重要です。 なぜなら、ユーザーがノードを実行することで、サードパーティにデータを供給してもらうのではなく、自身で暗号チェックを行って情報を検証できるからです。 また、ノードを実行することで、仲介者を介することなく、イーサリアムのピアツーピアネットワークに直接トランザクションを送信できます。 これらの利点を享受できるのが高価なハードウェアを使用するユーザーだけだと、分散化は実現できません。 そのため、ノードは、携帯電話やマイクロコンピュータ、家庭用コンピュータでも問題なく動作できるように、処理要件やメモリ要件を極力抑えて実行する必要があります。

現在のイーサリアムでは、ノードへのユニバーサルアクセスを妨げている主な障壁は、高いディスク容量要件です。 これは主に、イーサリアムの状態データの大部分を保存する必要があることに起因しています。 状態データには、新しいブロックとトランザクションを正しく処理するために必要となる重要な情報が含まれています。 この記事の執筆時点では、イーサリアムのフルノードを実行するには、2TBの高速SSDが推奨されています。 古いデータをプルーニングしないノードの場合、ストレージ容量は週に約14GBずつ増加していきます。誕生以降のすべてのデータを保存するアーカイブノードでは、現在12TB近くの容量が必要です(この記事は2023年2月に執筆されました)。

古いデータは安価なハードドライブで保存できますが、新しいこれから受信するブロックを処理するには遅すぎます。 イーサリアムの状態は「無限」に増えるため、現在のクライアントのストレージ設計を維持したまま、データをより安価で保存できるようにしても、問題の根本的な解決にはなりません。つまり、ストレージ要件は今後も増加する可能性があり、技術的な改善は常に求められます。また、状態の肥大化に追いつくためにも、継続的な努力が必要です。 そのため、クライアントは、ローカルデータベースによるデータの検索に依存しない、ブロックとトランザクションを検証する新しい方法を見つける必要があります。

## ノードのストレージ容量の削減 {#reducing-storage-for-nodes}

各ノードが保存しなければならないデータ量を減らす方法はいくつかあり、それぞれが異なる程度でイーサリアムのコアプロトコルの更新を必要とします。

- **履歴の有効期限**: イーサリアムクライアントの状態データを処理する方法自体は変更しませんが、ノードがXブロックよりも古い状態データを破棄できるようにします。
- **状態の有効期限**: 頻繁に使用されない状態データを非アクティブにすることができます。 非アクティブなデータは、復元されない限りクライアントによって無視されます。
- **弱いステートレス**: ブロック作成者のみが、すべての状態データにアクセスする必要があり、他のノードはローカル状態データベースがなくても、ブロックを検証できます。
- **強いステートレス**: すべての状態データにアクセスするノードが不要になります。

## データの有効期限 {#data-expiry}

### 履歴の有効期限 {#history-expiry}

履歴の有効期限とは、クライアントが必要性の低い古いデータを削除することです。少量の履歴データは少量しか保存しないため、新しいデータが到着すると古いデータを削除します。 クライアントが履歴データを必要とする理由は、2つあります。1つはデータの同期、もう1つはデータのリクエストの処理です。 もともとクライアントでは、始まりのブロックから同期することで、連続する各ブロックがチェーンの先頭に至るまで正しく追加されていることを検証する必要がありました。 しかし、現在のクライアントでは、「弱い主観性チェックポイント」と呼ばれる方法を使って、チェーンの先頭にたどり着くまでの時間を短縮しています。 これらのチェックポイントは、イーサリアムの始まりではなく、現在に近い始まりのブロックを基準にしているため、信頼できる開始点となります。 つまり、最新の弱い主観性チェックポイントより前のすべての情報は、 クライアントがチェーンの先頭へ同期する機能に影響を与えることなく削除できるということです。 現在のクライアントは、ローカルデータベースから履歴データを取得することで、(JSON-RPC経由で届く)履歴データのリクエストを処理しています。 ただし、履歴の有効期限が切れると、リクエストされたデータが削除されている場合は、履歴データを取得できません。 この履歴データを提供するには、いくつかの革新的なソリューションが必要です。

履歴データを取得する方法の1つは、クライアントがポータルネットワークなどのソリューションを介して、ピアから履歴データをリクエストすることです。 ポータルネットワークは、まだ開発中ではありますが、履歴データを提供するピアツーピアネットワークです。各ノードは、イーサリアムの履歴の一部を保存し、履歴全体がネットワーク全体に分散されるようにします。 履歴データのリクエストは、関連するデータを保存しているピアを探し出し、そのピアからデータを取得することで処理されます。 また、履歴データへのアクセスを必要とするのは、通常アプリであるため、アプリがデータを保存する役割を担うかもしれません。 イーサリアム空間には、履歴のアーカイブを維持したいという利他的なアクターが十分に存在する可能性があります。 履歴データストレージを管理するためのDAOが立ち上がる可能性もあります。あるいは、これらすべての選択肢を組み合わせた理想的な方法があるかもしれません。 これらのプロバイダは、トレント、FTP、Filecoin、IPFSなど、様々な方法でデータを提供することが考えられます。

履歴の有効期限を実装することについては、多少の議論があります。イーサリアムはこれまで、履歴データが常に利用可能であることを暗黙的に保証してきました。 そのため、誕生からのフル同期は、過去のデータの再構築にスナップショットを利用する場合でも、標準として常に可能になっています。 履歴の有効期限があることで、イーサリアムコアプロトコルは履歴データの保証責任を放棄します。 こうして、過去データを提供するのが中央集権的な組織になってしまうと、新たな検閲リスクが生じる可能性があります。

EIP-4444は、現在も活発な議論が行われており、まだリリースする準備はできていません。 EIP-4444の課題は、技術的なものではなく、そのほとんどがコミュニティ管理に関するものであることが興味深い点です。 この機能をリリースするには、単なる合意だけでなく、信頼できるエンティティによる履歴データの保存および提供の約束を含めたコミュニティの賛同が必要になります。

このアップグレードは、イーサリアムノードにおける状態データの扱いを大きく変えるものではありません。あくまでも、履歴データへのアクセス方法を変更するだけです。

### 状態の有効期限 {#state-expiry}

状態の有効期限とは、一定期間アクセスされていない状態をノードから削除することです。 これを実装する方法として、次のようなものがあります。

- **レンタル料による有効期限**: アカウントに「レンタル料」を請求し、レンタル料がゼロになると期限切れになります。
- **時間による有効期限**: 一定時間アカウントへの読み取り/書き込みがなければ、アカウントを非アクティブにします。

レンタル料による有効期限の場合、データベースをアクティブ状態に維持するためには、アカウントに直接貸し出すことが考えられます。 時間による有効期限の場合、最後のアカウント操作から有効期限をカウントダウンによるものか、すべてのアカウントの定期的な有効期限切れによるものか、いずれの可能性も考えられます また、時間ベースのモデルとレンタル料ベースのモデルの両方の要素を組み合わせたメカニズムも考えられます。例えば、各アカウントは、時間ベースの有効期限が切れる前に少額の料金を支払いをすることで、アクティブな状態を維持できる等です。 状態の有効期限が切れても、非アクティブな状態は**削除されない**ことに注意してください。アクティブな状態とは別に保存されます。 また、非アクティブな状態をアクティブに戻すこともできます。

この機能を実現するには、約1年間の状態ツリーが必要です。 新たな期間が開始するたびに、新たな状態ツリーが作成されます。 現在の状態ツリーのみ変更でき、以前のものは変更できません。 イーサリアムノードでは、現在の状態ツリーと次の最新の状態ツリーのみを保持する予定です。 そのためには、アドレスにその存在期間をタイムスタンプする方法が必要になります。 [いくつかの方法](https://ethereum-magicians.org/t/types-of-resurrection-metadata-in-state-expiry/6607)が考えられますが、有力な方法としては、追加情報を格納できるように[アドレスを長くするよう](https://ethereum-magicians.org/t/increasing-address-size-from-20-to-32-bytes/5485)要求することです。これにより、アドレスが長くなるほど安全性が高まるという利点も追加されます。 このロードマップアイテムは、[アドレス空間拡張](https://ethereum-magicians.org/t/increasing-address-size-from-20-to-32-bytes/5485)と呼ばれます。

履歴の有効期限と同様に、状態の有効期限では、ユーザーは古い状態データを自分で保存する必要がなくなります。代わりに、中央集権型のプロバイダー、利他的なコミュニティのメンバー、またはポータルネットワークなどのより革新的な分散型ソリューションなど、他のエンティティが保存の責任を担います。

状態の有効期限は、まだ研究段階であるため、リリースする準備ができていません。 状態の有効期限は、ステートレスクライアントや履歴の有効期限よりも後にリリースされるかもしれません。なぜなら、ステートレスクライアントや履歴の有効期限などのアップグレードによって、多数のバリデータが大きなサイズの状態を管理しやすくなるからです。

## ステートレス {#statelessness}

「状態」の概念がなくなるわけではなく、イーサリアムノードが状態データを処理する方法が変更されるものであるため、ステートレスという名称は少し不適切かもしれません。 ステートレスには、弱いステートレスと強いステートレスの2種類があります。 弱いステートレスでは、状態ストレージの役割を少数のノードに負わせることで、ほとんどのノードをステートレスにすることができます。 強いステートレスでは、すべてのノードが完全な状態データを保存する必要がなくなります。 弱いステートレスと強いステートレスのどちらも、通常のバリデータに次の利点をもたらします。

- ほぼ瞬時に同期できる
- ブロックを順不同で検証できる
- 非常に低いハードウェア要件(電話など)でノードが実行できる
- ディスクの読み取り/書き込みが不要なため、安価なハードドライブでもノードが実行できる
- イーサリアムの暗号技術の将来のアップグレードに対応できる

### 弱いステートレス {#weak-statelessness}

弱いステートレスでは、イーサリアムノードが状態変化の検証方法を変更します。ただし、ネットワーク上のすべてのノードで状態ストレージが完全に不要になるわけではありません。 弱いステートレスでは、代わりにブロック提案者が状態ストレージの役割を担います。ネットワーク上の他のすべてのノードは、完全な状態データを保存せずにブロックを検証します。

**弱いステートレスでは、ブロックの提案には、完全な状態データへのアクセスが必要になりますが、ブロックの検証では状態データは不要です。**

これを実現するには、イーサリアムクライアントに[バークルツリー](/roadmap/verkle-trees/)が実装されている必要があります。 バークルツリーは、イーサリアムの状態データを保存するための次世代のデータ構造です。ローカルデータベースでブロックを検証する代わりに、データに対して小さな固定サイズの「ウィットネス」をピア間で受け渡し、ブロックを検証します。 [プロポーザー/ビルダーセパレーション](/roadmap/pbs/)も必要です。これにより、ブロックビルダーは、完全な状態データへのアクセスが必要なため、より強力なハードウェアを備えた専門のノードになります。

<ExpandableCard title="少数のブロック提案者に依存するのはなぜでしょうか？" eventCategory="/roadmap/statelessness" eventName="clicked why is it OK to rely on fewer block proposers?">

ステートレスでは、ブロックの検証に必要なウィットネスを生成するのはブロックビルダーです。ブロックビルダーは、ブロックを検証するために必要なすべての状態データを保持しているため、 他のノードは状態データにアクセスする必要はありません。ブロックを検証するために必要な情報はすべてウィットネスが持っていることから、 ブロックの提案はコストが高く、ブロックの検証はコストが低いという状況になります。そのため、ブロックを提案するノードを実行するオペレーターが減ってしまう可能性があります。 しかし、ブロック提案者の分散化は、できるだけ多くの参加者が、提案されたブロックの有効性を独自で検証できる限り、重要ではありません。

<ButtonLink variant="outline-color" href="https://notes.ethereum.org/WUUUXBKWQXORxpFMlLWy-w#So-why-is-it-ok-to-have-expensive-proposers">詳細は、ダンクラットのメモをご覧ください。</ButtonLink>
</ExpandableCard>

ブロックの提案者は、状態データを使用して「ウィットネス」を作成します。これはブロック内のトランザクションによって変更される状態の値を証明する小さなデータセットです。 他のバリデータは状態を保持せず、状態ルート(状態全体のハッシュ)のみを保存します。 バリデータは、ブロックとウィットネスを受け取って、状態ルートを更新します。 こうすることで、バリデータノードは大幅に軽量化されます。

弱いステートレスについての研究は進んでいますが、、小さなウィットネスをピア間で受け渡しできるよう、プロポーザー/ビルダーセパレーションとバークルツリーの実装が前提になっています。 そのため、弱いステートレスがイーサリアムメインネットに実装されるのは、まだ数年先になると考えられています。

### 強いステートレス {#strong-statelessness}

強いステートレスでは、ノードに状態データを保存する必要がなくなります。 代わりに、集約されたウィットネスとともにトランザクションが送信されます。 ブロック生成者は、関連するアカウントのウィットネスを生成するために必要な状態のみを保存します。 状態に対する責任はほとんどユーザーが負い、ユーザーはどのアカウントとストレージ鍵とやり取りしているかを宣言するために、ウィットネスと「アクセスリスト」を送信します。 これにより、ノードが非常に軽量になりますが、スマートコントラクトとのトランザクションがより困難になるなどのトレードオフがあります。

強いステートレスは、研究者によって調査されていますが、現時点では、イーサリアムのロードマップには含まれていません。イーサリアムのスケーリングには、弱いステートレスが十分に機能すると考えられているためです。

## 現在の進行状況 {#current-progress}

弱いステートレス、履歴の有効期限、状態の有効期限は、すべて研究段階にあります。リリースされるのは数年後になる予定ですが、 これらの提案がすべて実装されるわけではありません。例えば、状態の有効期限が最初に実装された場合、履歴の有効期限の実装は不要になるかもしれません。 また、[バークルツリー](/roadmap/verkle-trees)や[プロポーザー/ビルダーセパレーション](/roadmap/pbs) など、先に完了しなければならない他のロードマップアイテムもあります。

## 参考文献 {#further-reading}

- [ヴィタリックによるステートレスに関するAMA](https://www.reddit.com/r/ethereum/comments/o9s15i/impromptu_technical_ama_on_statelessness_and/)
- [状態サイズの管理理論](https://hackmd.io/@vbuterin/state_size_management)
- [Resurrection-conflict-minimized 状態境界](https://ethresear.ch/t/resurrection-conflict-minimized-state-bounding-take-2/8739)
- [ステートレスと状態の有効期限への工程](https://hackmd.io/@vbuterin/state_expiry_paths)
- [EIP-4444の仕様](https://eips.ethereum.org/EIPS/eip-4444)
- [アレックス・ストークス(Alex Stokes)EIP-4444の概要を説明するビデオ](https://youtu.be/SfDC_qUZaos)
- [ステートレスにすることが重要な理由](https://dankradfeist.de/ethereum/2021/02/14/why-stateless.html)
- [ステートレスクライアントのオリジナルコンセプトに関するノート](https://ethresear.ch/t/the-stateless-client-concept/172)
- [状態の有効期限の詳細](https://hackmd.io/@vbuterin/state_size_management#A-more-moderate-solution-state-expiry)
- [状態の有効期限のさらなる詳細](https://hackmd.io/@vbuterin/state_expiry_paths#Option-2-per-epoch-state-expiry)
