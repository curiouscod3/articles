<sub>&#x1F6A8; <strong>Autogenerated!</strong> See <a href="https://github.com/ponyfoo/articles/tree/noindex/contributing.markdown"><code>contributing.markdown</code></a> for details. See also: <a href="https://ponyfoo.com/articles/where-does-this-keyword-come-from">web version</a>.</sub>

<a href="https://ponyfoo.com/articles/where-does-this-keyword-come-from"><div></div></a>

<h1>Where does this keyword come from?</h1>

<p><kbd>js</kbd> <kbd>scoping</kbd> <kbd>best-practices</kbd> <kbd>this</kbd></p>

<blockquote><p>Working on the latest chapter for my upcoming book on <a href="http://bevacqua.io/buildfirst" target="_blank">JavaScript Application Design</a>, I&#x2019;m writing about how scoping works. I want to share a particular code sample &#x2026;</p></blockquote>

<div><p>Working on the latest chapter for my upcoming book on <a href="http://bevacqua.io/buildfirst" target="_blank">JavaScript Application Design</a>, I&#x2019;m writing about how scoping works. I want to share a particular code sample which I hope will bring some clarity to how <code class="md-code md-code-inline">this</code> works. It&#x2019;s not all dark magic, learning about <code class="md-code md-code-inline">this</code> can be <em>tremendously helpful</em> to your development as a JavaScript programmer.</p></div>

<blockquote></blockquote>

<div><p>Until you <em>&#x201C;get it&#x201D;</em>, this is probably how you feel about <code class="md-code md-code-inline">this</code>.</p> <figure class="figure-has-loaded"><img src="https://raw.github.com/bevacqua/buildfirst/master/images/chaos.gif" alt="chaos.gif"></figure> <p>It&#x2019;s madness, right? In this brief article, I aim to demystify <code class="md-code md-code-inline">this</code>.</p></div>

<div><h2 id="how-this-works">How <code class="md-code md-code-inline">this</code> works</h2> <p>If the method is invoked on an object, that object will be assigned to <code class="md-code md-code-inline">this</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> parent = {
    method: <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
        <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">this</span>);
    }
};

parent.method();
<span class="md-code-comment">// &lt;- parent</span>
</code></pre> <p>Note that this behavior is very <em>&#x201C;fragile&#x201D;</em>, if you get a reference to method, and invoke that, then <code class="md-code md-code-inline">this</code> won&#x2019;t be <code class="md-code md-code-inline">parent</code> anymore, but rather the <code class="md-code md-code-inline">window</code> global object once again. This confuses most developers.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> parentless = parent.method;

parentless();
<span class="md-code-comment">// &lt;- Window</span>
</code></pre> <p>The bottom line is you should look at the call site to figure out whether the function is invoked as a property of an object or on its own. If its invoked as a property, then that property will become <code class="md-code md-code-inline">this</code>, otherwise <code class="md-code md-code-inline">this</code> will be assigned the value of the global object, or <code class="md-code md-code-inline">window</code>. In this case, but under <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode" target="_blank" aria-label="Strict mode explained on MDN">strict mode</a>, <code class="md-code md-code-inline">this</code> will be <code class="md-code md-code-inline">undefined</code> instead.</p> <p>In the case of constructor functions, <code class="md-code md-code-inline">this</code> is assigned to the instance that&#x2019;s being created, when using the <code class="md-code md-code-inline">new</code> keyword.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">ThisClownCar</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">this</span>);
}

<span class="md-code-keyword">new</span> ThisClownCar();
<span class="md-code-comment">// &lt;- ThisClownCar {}</span>
</code></pre> <p>Note that this behavior doesn&#x2019;t have a way of telling a function is supposed to be used as a constructor function, and thus omitting the new keyword will result in <code class="md-code md-code-inline">this</code> being the global object, like we saw in the <code class="md-code md-code-inline">parentless</code> example.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">ThisClownCar();
<span class="md-code-comment">// &lt;- Window</span>
</code></pre> <h2 id="tampering-with-this">Tampering with <code class="md-code md-code-inline">this</code></h2> <p>The <code class="md-code md-code-inline">.call</code>, <code class="md-code md-code-inline">.apply</code>, and <code class="md-code md-code-inline">.bind</code> methods are used to manipulate function invocation, helping us to define both the value for <code class="md-code md-code-inline">this</code>, and the <code class="md-code md-code-inline">arguments</code> provided to the function.</p> <p><code class="md-code md-code-inline">Function.prototype.call</code> takes any number of arguments, the first one is assigned to <code class="md-code md-code-inline">this</code>, and the rest are passed as arguments to the function that&#x2019;s being invoked.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.slice.call([<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">3</span>], <span class="md-code-number">1</span>, <span class="md-code-number">2</span>)
<span class="md-code-comment">// &lt;- [2]</span>
</code></pre> <p><code class="md-code md-code-inline">Function.prototype.apply</code> behaves very similarly to <code class="md-code md-code-inline">.call</code>, but it takes the arguments as a single array with every value, instead of any number of parameter values.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">String</span>.prototype.split.apply(<span class="md-code-string">&apos;13.12.02&apos;</span>, [<span class="md-code-string">&apos;.&apos;</span>])
<span class="md-code-comment">// &lt;- [&apos;13&apos;, &apos;12&apos;, &apos;02&apos;]</span>
</code></pre> <p><code class="md-code md-code-inline">Function.prototype.bind</code> creates a special function which can be used to invoke the function it is called on. That function will always use the <code class="md-code md-code-inline">this</code> argument passed to <code class="md-code md-code-inline">.bind</code>, as well as being able to assign a few arguments, creating a <a href="http://en.wikipedia.org/wiki/Currying" target="_blank" aria-label="Currying on Wikipedia">curried version</a> of the original function.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> arr = [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>];
<span class="md-code-keyword">var</span> add = <span class="md-code-built_in">Array</span>.prototype.push.bind(arr, <span class="md-code-number">3</span>);

<span class="md-code-comment">// effectively the same as arr.push(3)</span>
add();

<span class="md-code-comment">// effectively the same as arr.push(3, 4)</span>
add(<span class="md-code-number">4</span>);

<span class="md-code-built_in">console</span>.log(arr);
<span class="md-code-comment">// &lt;- [1, 2, 3, 3, 4]</span>
</code></pre> <h2 id="scoping-this">Scoping <code class="md-code md-code-inline">this</code></h2> <p>In the next case, <code class="md-code md-code-inline">this</code> will stay the same across the scope chain, this is the exception to the rule, and often leads to confusion among amateur developers.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">scoping</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">this</span>);

  <span class="md-code-keyword">return</span> <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">this</span>);
  };
}

scoping()();
<span class="md-code-comment">// &lt;- Window</span>
<span class="md-code-comment">// &lt;- Window</span>
</code></pre> <p>A common work-around is to create a local variable which holds onto the reference to <code class="md-code md-code-inline">this</code>, and isn&#x2019;t shadowed in the child scope. The child scope shadows <code class="md-code md-code-inline">this</code>, making it impossible to access a reference to the parent <code class="md-code md-code-inline">this</code> directly.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">retaining</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">var</span> self = <span class="md-code-keyword">this</span>;

  <span class="md-code-keyword">return</span> <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-built_in">console</span>.log(self);
  };
}

retaining()();
<span class="md-code-comment">// &lt;- Window</span>
</code></pre> <p>Unless you really want to use both the parent scope&#x2019;s <code class="md-code md-code-inline">this</code>, as well as the current value of <code class="md-code md-code-inline">this</code> for some obscure reason, the method I prefer is to use the <code class="md-code md-code-inline">.bind</code> function. This can be used to assign the parent <code class="md-code md-code-inline">this</code> to the child scope.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">bound</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">this</span>);
  }.bind(<span class="md-code-keyword">this</span>);
}

bound()();
<span class="md-code-comment">// &lt;- Window</span>
</code></pre> <p>Have you ever had any problems with this? How about <code class="md-code md-code-inline">this</code>? Let me know if you think I&#x2019;ve missed any other edge cases or elegant solutions.</p></div>
