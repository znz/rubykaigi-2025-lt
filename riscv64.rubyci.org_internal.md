# riscv64.rubyci.org internal

author
:   Kazuhiro NISHIYAMA

institution
:   株式会社Ruby開発

content-source
:   RubyKaigi 2025 LT

date
:   2025-04-17

allotted-time
:   5m

theme
:   lightning-simple

# What's rubyci.org?

- *CI* results summary site
  - <https://rubyci.org/>
- Run *chkbuild* on each CI environments
  - <https://github.com/ruby/chkbuild>
- Most environments run on *all* supported ruby versions
- Some environments run on *master branch only*
  - jit variants, android, *riscv64*

## note

まず、rubyci.org とは、CIの結果のまとめサイトです。
各 CI 環境で chkbuild を実行しています。
ほとんどの環境では現在サポートされている ruby の全バージョンを実行しています。
いくつかの環境ではマスターブランチのみ実行しています。

# Why do I maintain riscv64 VM?

- I am interested in *minor environments*, and run tests on them.
  - It may find interesting bugs.
- `qemu-system-riscv64` is easy to run lately.
- When I took over the riscv64 VM,
  there was an environment created by mame-san,
  but it was a bit out of date.

## note

risc 環境のメンテナンスをしている理由は、マイナーな環境でテストを実行するとバグがみつかるかもしれないからです。
最近は qemu で実行しやすくなっているのも理由のひとつです。
まめさんが作成した環境がすでにあったのですが、少し古くなっていたので引き継いでメンテナンスしています。

# Premise

- `qemu-system-riscv64` runs on host environments, and *very slow*.
  - CPU emulation using qemu-system takes longer than running on a real machine.
- So it runs *master branch only*.
- Nevertheless it can *take hours*.

As a result, rebooting normally would *interrupt CI*, which would be *a huge waste*, so I devised a way to reboot between CI runs.

## note

前提として、qemu 環境は非常に遅いので、マスターブランチのみ実行しています。
それでも、7時間ぐらいかかっています。

そのため、再起動で CI が中断すると非常に無駄が大きいので、CI の実行の合間に再起動するように工夫しました。

# How to run without interrupt

- `unattended-upgrade` sometimes requires reboot
  after upgrading packages.
  - It creates `/run/reboot-required`.
- I take an hour to maintain between running chkbuild.
  - Machine reboots on that time if required.

## note

パッケージの更新後に再起動が必要になると `unattended-upgrade` が `/run/reboot-required` を作成します。
`chkbuild` の実行の合間にメンテナンス時間をとっていて、必要ならそこで再起動します。

# Guest VM and Host OS

- It may easily wait for the guest VM to reboot itself.
- It should wait for rebooting the host OS too.

## note

これでゲストVM自体の再起動は簡単に待てるかもしれませんが、
ホストOSの再起動も待つ必要があります。

# How to wait?

- They cooperate using a shared directory.
  - Update `mtime` of the specific file in it after chkbuild finished.
  - My own systemd path unit detects changing `mtime`,
    and reboot if required on both guest and host.
- Pros
  - They can be loose coupling.
  - Notifier has less privileges.

## note

そこで、共有ディレクトリを使って待つようにしました。
具体的にはchkbuildが終わったときに、特定のファイルの更新時刻を更新して、
systemd の path unit で検出して、必要なら再起動します。
この方法は、疎結合にできるのと、通知側の権限を少なくできる、というのが利点です。

# Do you interested?

- If you want ruby to support your favorite environments more,
  - you can maintain your own CI environment to run chkbuild
    - Run `start-rubyci` in `ruby/chkbuild`
  - and add results to `rubyci.org`
	- Contact to `rubyci.org` maintainer to add URLs of your chkbuild output

## note

こんな感じで、好きな環境でのrubyをもっとサポートしたいと思ったら、
chkbuild を動かす環境をととのえて、 rubyci.org に追加してもらうと良いでしょう。

# self.introduction

- Kazuhiro NISHIYAMA
- One of the Ruby Committers
- github, etc.: `@znz`
- 株式会社Ruby開発 www.ruby-dev.jp
  - We are hiring!

## note

最後に自己紹介です。
Ruby コミッターのひとりで、github などはゼットエヌゼットというアカウントで活動しています。
株式会社Ruby開発に所属しています。採用強化中なので興味があればよろしくお願いします。
以上で発表を終わります。
