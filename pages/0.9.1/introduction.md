# Introduction

Geb is a developer focused tool for automating the interaction between web browsers and web content. It uses the dynamic language features of Groovy to provide a powerful content definition DSL (for modelling content for reuse) and key concepts from jQuery to provide a powerful content inspection and traveral API (for finding and interacting with content).

Geb(ジェブ)は開発者視点のツールで、WebブラウザーとWebコンテンツ間の統合を自動で行うことができます。Gebで使われているのは、ダイナミック[Groovy]を利用しており、協力なコンテンツ定義DSL（再利用コンテンツモデリング）を提供し、[jQuery]からのキーコンセプトとして、強力なコンテンツ抽出、traveral API(コンテンツの検索、アクセス)を獲得している。

Geb was born out of a desire to make browser automation (originally for web testing) easier and more productive. It aims to be a developer tool in that it allows and encourages the using of programming and language constructs instead of creating a restricted environment. It uses Groovy’s dynamism to remove the noise and boiler plate code in order to focus on what’s important — the content and interaction.

Gebはブラウザ自動化（元々はWebテストとして）を簡単に、より生産性あるものとして実行することを求められた背景で生まれた。目的は開発ツールであり、プログラミングすることを奨励し、言語構造は特定の環境で作られたにもかかわらず。GebはGroovyのダイナミックエンジンを利用し、ノイズを除去し、より重要な、コンテンツとそのアクセスに焦点を当てること重視している。

## ブラウザ自動化技術
Gebは[WebDriver]というブラウザ操作の自動化ライブラリをベースとしている。
つまり、Gebは[WebDriverが動作するあらゆるブラウザ]で動作することが可能である。
Gebでは。使い勝手がよく生産的な外部レイヤーを提供することでWebDeriverを扱いやすくし、ユーザが必要なことを常に直接的に行うことが可能となっている。

さらに情報が必要な場合は、[driver実装方法]のマニュアルセクションを参照とする。


## Pageオブジェクトパターン

Pageオブジェクトパターンは、再利用性、メンテンナンス性を達成するためのモデルケースとなる。
WebDriverのwikiページを引用すると、Pageオブジェクトパターンとは：

>Within your web app’s UI there are areas that your tests interact with. A Page Object simply models these as objects within the test code. This reduces the amount of duplicated code and means that if the UI changes, the fix need only be applied in one place.

>WebアプリケーションのUIにおいてテストしたい箇所があるとする。Pageオブジェクトがモデリングしているのは、テストコードに含まれるオブジェクトそのものとなる。これにより複雑なコード量が減り、UIが変更されても、テストの修正はその変更されたオブジェクト部分のみでよい。

さらには（上記と同じドキュメントより）：

>PageObjects can be thought of as facing in two directions simultaneously. Facing towards the developer of a test, they represent the services offered by a particular page. Facing away from the developer, they should be the only thing that has a deep knowledge of the structure of the HTML of a page (or part of a page) It’s simplest to think of the methods on a Page Object as offering the “services” that a page offers rather than exposing the details and mechanics of the page. As an example, think of the inbox of any web-based email system. Amongst the services that it offers are typically the ability to compose a new email, to choose to read a single email, and to list the subject lines of the emails in the inbox. How these are implemented shouldn’t matter to the test.

> ？？Pageオブジェクトは、2つの方向性を同時に対応することを考慮されている。テスト開発者に向けたものであるが、テスト開発者は特定のページを提供されているサービスを要求する。開発者ではない人においては、WebページのHTML構造に深い知識が
最もシンプルな考えとして、Pageオブジェクトはサービスを提供している。サービスとは、Webページの詳細やメカニズムを詳らかにすること無く提供するものである。サービスではそれを提供することで、メールボックスにおいて、新しいメールを作成したり、あるメールを読むために選択したり、メールのタイトルを一覧にしたりすることができる。どのように実装するかをテストで心配する必要はないのである。

Pageオブジェクトパターンは重要な技術であり、よって、GebではPageとModule構造を通して最優先的にサポートしている。


## jQueryライクAPI
Javascriptライブラリの１つである[jQuery]は素晴らしいAPIを提供しており、ページ内のコンテンツを検索、選択し、コンテンツを横断的に扱うことができる。Gebはこの考えに多くのインスピレーションを得ている。

Gebにおいて、コンテンツは$関数を通して選択され、[Navigator]オブジェクトを通して返却される。Navigatorオブジェクトは、jQueryにおけるjQueryデータ形式に似ており、ページ内の1つまたは複数のエレメントを表現している。

いくつかのサンプルを見てみよう。

<pre><code>// match all 'div' elements on the page
$("div")
 
// match the first 'div' element on the page
$("div", 0)
 
// match all 'div' elements with a title attribute value of 'section'
$("div", title: "section")
 
// match the first 'div' element with a title attribute value of 'section'
$("div", 0, title: "section")
 
// match all 'div' elements who have the class 'main'
$("div.main") 
 
// match the first 'div' element with the class 'main'
$("div.main", 0) 
</code></pre>
これらのメソッドでは、コンテンツをより洗練した利用の仕方ができるようなNavigatorオブジェクトを返す。

<pre><code>// The parent of the first paragraph
$("p", 0).parent()
 
// All tables with a cellspacing attribute value of 0 that are nested in a paragraph
$("p").find("table", cellspacing: '0')
</code></pre>
これがNavigatorAPIの初歩である。Navigatorに関して詳しく知りたい場合は、[Navigator]の項目を参照とする。


## サンプルソース
それでは、Googleで"wikipedia"を検索し、最初の検索結果を選択する単純なケースを見てみよう。

### インラインスクリプト
Gebを利用したインライン（ここではPageオブジェクトや定義済みコンテンツではないことを示す）スクリプトのサンプルを示す。

<pre><code>import geb.Browser
 
Browser.drive {
    go "http://google.com/"
 
    // make sure we actually got to the page
    assert title == "Google"
 
    // enter wikipedia into the search field
    $("input", name: "q").value("wikipedia")
 
    // wait for the change to results page to happen
    // (google updates the page dynamically without a new request)
    waitFor { title.endsWith("Google Search") }
 
    // is the first link to wikipedia?
    def firstLink = $("li.g", 0).find("a.l")
    assert firstLink.text() == "Wikipedia"
 
    // click the link 
    firstLink.click()
 
    // wait for Google's javascript to redirect to Wikipedia
    waitFor { title == "Wikipedia" }
}
</code></pre>


### Pageオブジェクトを利用したソースコード
今度はPageオブジェクトパターンを利用して、事前にコンテンツを定義してみよう。

<pre><code>import geb.Browser
import geb.Page
import geb.Module
 
// modules are reusable fragments that can be used across pages that can be paramaterised
// here we are using a module to model the search function on the home and results pages
class GoogleSearchModule extends Module {
 
    // a paramaterised value set when the module is included
    def buttonValue
 
    // the content DSL
    static content = {
 
        // name the search input control “field”, defining it with the jQuery like navigator
        field { $("input", name: "q") }
 
        // the search button declares that it takes us to the results page, and uses the 
        // parameterised buttonValue to define itself
        button(to: GoogleResultsPage) { 
            $("input", value: buttonValue)
        }
    }
}
 
class GoogleHomePage extends Page {
 
    // pages can define their location, either absolutely or relative to a base
    static url = "http://google.com/ncr"
 
    // “at checkers” allow verifying that the browser is at the expected page
    static at = { title == "Google" }
 
    static content = {
        // include the previously defined module
        search { module GoogleSearchModule, buttonValue: "Google Search" }
    }
}
 
class GoogleResultsPage extends Page {
    static at = { title.endsWith "Google Search" }
    static content = {
        // reuse our previously defined module
        search { module GoogleSearchModule, buttonValue: "Search" }
 
        // content definitions can compose and build from other definitions
        results { $("li.g") }
        result { i -> results[i] }
        resultLink { i -> result(i).find("a.l") }
        firstResultLink { resultLink(0) }
    }
}
 
class WikipediaPage extends Page {
    static at = { title == "Wikipedia" }
}
</code></pre>

定義したコンテンツを利用すると、下記のようになる。

<pre><code>Browser.drive {
    to GoogleHomePage
    assert at(GoogleHomePage)
    search.field.value("wikipedia")
    waitFor { at GoogleResultsPage }
    assert firstResultLink.text() == "Wikipedia"
    firstResultLink.click()
    waitFor { at WikipediaPage }
}
</code></pre>


### テストコード
Geb自体はテストフレームワークを含んでいない。その代わり、すでに存在する有名なテストフレームワークである、[Spock]、[JUnit]、[TestNG]、[Cucumber]と[EasyB]にて利用することができる。Gebは様々なテストフレームワークにて利用することができるが、GebのスタイルによくマッチするSpockを推奨している。

Googleのケースを再び利用して、GebとSpockを組み合わせた形で示す。

<pre><code>import geb.spock.GebSpec
 
class GoogleWikipediaSpec extends GebSpec {
 
    def "first result for wikipedia search should be wikipedia"() {
        given:
        to GoogleHomePage
 
        expect:
        at GoogleHomePage
 
        when:
        search.field.value("wikipedia")
 
        then:
        waitFor { at GoogleResultsPage }
 
        and:
        firstResultLink.text() == "Wikipedia"
 
        when:
        firstResultLink.click()
 
        then:
        waitFor { at WikipediaPage }
    }
}
</code></pre>

For more information on using Geb for web and functional testing, see the testing chapter.
Gebを利用してたWebの機能テストに関して詳しく知りたい場合は、[テストチャプター]を参照とする。

## インストール方法＆使用方法
Geb自体は、[GebのMavenリポジトリ]から取得可能なgeb-coreのjarファイルにまとめられている。あとは、このjarファイルに加えて、WebDriverとselenium-supportのjarファイルがあれば使用することが可能である。


@Grabでは...

<pre><code>@Grapes([
    @Grab("org.gebish:geb-core:0.9.1"),
    @Grab("org.seleniumhq.selenium:selenium-firefox-driver:2.26.0"),
    @Grab("org.seleniumhq.selenium:selenium-support:2.26.0")
])
import geb.Browser
</code></pre>

Mavenでは...

<pre><code>&lt;dependency>
  &lt;groupId>org.gebish&lt;/groupId>
  &lt;artifactId>geb-core&lt;/artifactId>
  &lt;version>0.9.1&lt;/version>
&lt;/dependency>
&lt;dependency>
  &lt;groupId>org.seleniumhq.selenium&lt;/groupId>
  &lt;artifactId>selenium-firefox-driver&lt;/artifactId>
  &lt;version>2.26.0&lt;/version>
&lt;/dependency>
&lt;dependency>
  &lt;groupId>org.seleniumhq.selenium&lt;/groupId>
  &lt;artifactId>selenium-support&lt;/artifactId>
  &lt;version>2.26.0&lt;/version>
&lt;/dependency>
</code></pre>

Gradleでは...

<pre><code>compile "org.gebish:geb-core:0.9.1", "org.seleniumhq.selenium:selenium-firefox-driver:2.26.0", "org.seleniumhq.selenium:selenium-support:2.26.0"
</code></pre>
一方、geb-spockやgeb-unitといった統合されたライブラリは、geb-coreの代わりとして利用することもできる。

>Grailsのような特定の環境でGebを利用する場合には、[Build Integrations]の章を確認してください。

[Groovy]:http://groovy.codehaus.org/ "Groovy"
[Webdriver]:http://code.google.com/p/selenium/ "Webdriver"
[jQuery]:http://jquery.com/ "jQuery"
[driver実装方法]:http://www.gebish.org/manual/0.9.1/driver.html "driver実装方法"
[WebDriverが動作するあらゆるブラウザ]:http://code.google.com/p/selenium/wiki/FrequentlyAskedQuestions#Q:_Which_browsers_does_support? "WebDriverが動作するブラウザ"
[Navigator]:http://www.gebish.org/manual/0.9.1/api/geb/navigator/Navigator.html "Navigator"
[Spock]:http://spockframework.org/
[JUnit]:http://www.junit.org/
[TestNG]:http://testng.org/
[Cucumber]:https://github.com/cucumber/cuke4duke/wiki
[EasyB]:http://www.easyb.org/
[テストチャプター]:
[GebのMavenリポジトリ]:http://mvnrepository.com/artifact/org.gebish/geb-core
[Build Integrations]: