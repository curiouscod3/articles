<sub>&#x1F6A8; <strong>Autogenerated!</strong> See <a href="https://github.com/ponyfoo/articles/tree/noindex/contributing.markdown"><code>contributing.markdown</code></a> for details. See also: <a href="https://ponyfoo.com/articles/observables-coming-to-ecmascript">web version</a>.</sub>

<a href="https://ponyfoo.com/articles/observables-coming-to-ecmascript"><div><img src="https://i.imgur.com/GKoh78o.jpg" alt="Observables Proposal for ECMAScript!"></div></a>

<h1>Observables Proposal for ECMAScript!</h1>

<p><kbd>ecmascript</kbd> <kbd>proposal-draft</kbd></p>

<blockquote><p>There&#x2019;s an ECMAScript proposal for Observables in the works. In this article we explore the proposal, the API, and look at a few use cases.</p><p>Observables in &#x2026;</p></blockquote>

<div><p>There&#x2019;s an ECMAScript proposal for Observables in the works. In this article we explore the proposal, the API, and look at a few use cases.</p></div>

<blockquote></blockquote>

<div><p>Observables in JavaScript were largely popularized by libraries such as <a href="https://github.com/Reactive-Extensions/RxJS" target="_blank" aria-label="Reactive-Extensions/RxJS on GitHub">RxJS</a> and <a href="https://baconjs.github.io/" target="_blank" aria-label="A small functional reactive programming lib for JavaScript">Bacon.js</a>. Jafar Husain, a Netflix tech lead and long-time functional programming advocate who&#x2019;s also on TC39 has been developing a proposal to bring observables into the core language. The <a href="https://github.com/tc39/proposal-observable" target="_blank" aria-label="tc39/proposal-observable on GitHub"><code class="md-code md-code-inline">Observable</code> proposal</a> is at stage 1 but marked as ready to move up to stage 2, at the time of this writing.</p></div>

<div><h1 id="observable-and-the-observer-api"><code class="md-code md-code-inline">Observable</code> and the <code class="md-code md-code-inline">observer</code> API</h1> <p>In this proposal, <code class="md-code md-code-inline">Observable</code> would be a new built-in that can be used to handle event streams. The <code class="md-code md-code-inline">Observable</code> constructor takes a callback which defines an event stream. In the following example, our observable returns a stream of events with just <code class="md-code md-code-inline">1</code> and <code class="md-code md-code-inline">2</code> values. The <code class="md-code md-code-inline">observer.next</code> method can be used to add events to the observable stream.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">new</span> Observable(observer =&gt; {
  observer.next(<span class="md-code-number">1</span>)
  observer.next(<span class="md-code-number">2</span>)
})
</code></pre> <p>We can use <code class="md-code md-code-inline">observer.error</code> to report errors that occur during stream processing.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">new</span> Observable(observer =&gt; {
  observer.error(<span class="md-code-keyword">new</span> <span class="md-code-built_in">Error</span>(`Failed to stream events`))
})
</code></pre> <p>We can use <code class="md-code md-code-inline">observer.complete</code> to signal when the event stream has come to an end.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">new</span> Observable(observer =&gt; {
  observer.next(<span class="md-code-number">1</span>)
  observer.next(<span class="md-code-number">2</span>)
  observer.complete()
})
</code></pre> <p>The callback passed to an <code class="md-code md-code-inline">Observable</code> constructor can return a cleanup function to tear down our observable. This can be useful to remove event listeners, clear timeouts, and similar cleanup tasks. The following observable is a bit more interesting than the last one. It tracks mouse position relative to the page as the user moves the cursor on screen, producing an event stream that describes the cursor position on the page through time.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">mouseTracking</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Observable(observer =&gt; {
    <span class="md-code-keyword">const</span> handler = ({ pageX, pageY }) =&gt; {
      observer.next({ x: pageX, y: pageY })
    }

    <span class="md-code-built_in">document</span>.body.addEventListener(`mousemove`, handler)

    <span class="md-code-keyword">return</span> () =&gt;  {
      <span class="md-code-built_in">document</span>.body.removeEventListener(`mousemove`, handler)
    }
  })
}
</code></pre> <p>In order for us to subscribe to an observable event stream, we just call the <code class="md-code md-code-inline">Observable#subscribe</code> method on an observable object instance. Doing so will invoke the callback passed to the <code class="md-code md-code-inline">Observable</code> constructor in the previous code snippet, binding the event listener and getting the event stream started. Moving the mouse will now result in events being fired into the event stream.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">mouseTracking().subscribe({
  next({ x, y }) { <span class="md-code-built_in">console</span>.log(`New position: ${ x }, ${ y }`) },
  error(err) { <span class="md-code-built_in">console</span>.log(`<span class="md-code-built_in">Error</span>: ${ err }`) },
  complete() { <span class="md-code-built_in">console</span>.log(`Done!`) }
})
</code></pre> <h1 id="subscription-unsubscribe"><code class="md-code md-code-inline">Subscription#unsubscribe</code></h1> <p><code class="md-code md-code-inline">Observable#subscribe</code> returns an object that lets us <code class="md-code md-code-inline">unsubscribe</code>, executing the cleanup method <em>&#x2013; if one exists</em>. When we&#x2019;re no longer interested in events from the observable stream, we&#x2019;ll unsubscribe and let the observable clean itself up.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">const</span> subscription = mouseTracking().subscribe({
  next({ x, y }) { <span class="md-code-built_in">console</span>.log(`New position: ${ x }, ${ y }`) },
  error(err) { <span class="md-code-built_in">console</span>.log(`<span class="md-code-built_in">Error</span>: ${ err }`) },
  complete() { <span class="md-code-built_in">console</span>.log(`Done!`) }
})

subscription.unsubscribe()
</code></pre> <h1 id="observableof"><code class="md-code md-code-inline">Observable.of</code></h1> <p><code class="md-code md-code-inline">Observable.of(...items)</code> is a simple static utility helper that creates an <code class="md-code md-code-inline">Observable</code> out of the provided <code class="md-code md-code-inline">items</code>. The <code class="md-code md-code-inline">items</code> are then delivered synchronously when <code class="md-code md-code-inline">Observable#subscribe</code> is called.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">Observable.of(<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">3</span>, <span class="md-code-number">4</span>).subscribe({
  next(item) { <span class="md-code-built_in">console</span>.log(item) }
})
<span class="md-code-comment">// &lt;- 1</span>
<span class="md-code-comment">// &lt;- 2</span>
<span class="md-code-comment">// &lt;- 3</span>
<span class="md-code-comment">// &lt;- 4</span>
</code></pre> <p>We can think of <code class="md-code md-code-inline">Observable.of</code> as the following <em>simplified</em> implementation, where we return a synchronous stream of provided values.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">Observable.of = (...items) =&gt; {
  <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Observable(observer =&gt; {
    items.forEach(item =&gt; {
      observer.next(item)
    })
    observer.complete()
  })
}
</code></pre> <h1 id="observablefrom"><code class="md-code md-code-inline">Observable.from</code></h1> <p>This static method casts the provided argument into an <code class="md-code md-code-inline">Observable</code>. If the provided <code class="md-code md-code-inline">Object</code> has a <code class="md-code md-code-inline">Symbol.observable</code> method, then the result of invoking that method is returned.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">Observable
  .from({
    [Symbol.observable]() { <span class="md-code-keyword">return</span> Observable.of(<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">3</span>) }
  })
  .subscribe({
    next(item) { <span class="md-code-built_in">console</span>.log(item) }
  })
<span class="md-code-comment">// &lt;- 1</span>
<span class="md-code-comment">// &lt;- 2</span>
<span class="md-code-comment">// &lt;- 3</span>
</code></pre> <p>If the provided argument doesn&#x2019;t implement a <code class="md-code md-code-inline">Symbol.observable</code> method, then it&#x2019;s assumed to be an iterable. A synchronous <code class="md-code md-code-inline">Observable</code> sequence of the iterable is returned.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">Observable
  .from([<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">3</span>])
  .subscribe({
    next(item) { <span class="md-code-built_in">console</span>.log(item) }
  })
<span class="md-code-comment">// &lt;- 1</span>
<span class="md-code-comment">// &lt;- 2</span>
<span class="md-code-comment">// &lt;- 3</span>
</code></pre> <p>In this case, <code class="md-code md-code-inline">Observable.from</code> is similar to <code class="md-code md-code-inline">Observable.of</code>. We could think of <code class="md-code md-code-inline">Observable.from</code> as the following <em>simplified</em> implementation.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">Observable.from = value =&gt; {
  <span class="md-code-keyword">if</span> (<span class="md-code-keyword">typeof</span> value[Symbol.observable] === `<span class="md-code-function"><span class="md-code-keyword">function</span>`) </span>{
    <span class="md-code-keyword">return</span> value[Symbol.observable]()
  }
  <span class="md-code-keyword">return</span> Observable.of(<span class="md-code-built_in">Array</span>.from(value))
}
</code></pre> <h1 id="conclusions">Conclusions</h1> <p>Note that this proposal is still in its infancy, but it&#x2019;d lay out the foundation for functional programming at the native JavaScript level. Eventually, it may earn the ability to <code class="md-code md-code-inline">Observable#filter</code> or <code class="md-code md-code-inline">Observable#map</code> the stream of events, allowing us to focus only on the kinds of events we want to listen for.</p> <p>In the meantime, these could be implemented in user-land as we continue to watch out for patterns and let the specification evolve naturally and gradually. You can find <a href="https://github.com/tc39/proposal-observable/blob/0fa13995f372bab50de8cb5e8db59066ad08dd7a/src/Observable.js" target="_blank" aria-label="Observable.js polyfill on GitHub">a polyfill for the current incarnation</a> of the specification on the GitHub repository, but you&#x2019;ll have to delete the <code class="md-code md-code-inline">export</code> keyword if you want to play around with it in your browser&#x2019;s Dev Tools.</p></div>
