---
tabId: readable
tabLabel: Readable Code
---
<div class="scala-in-action-content">
  <div class="scala-in-action-code">
    <div class="wrap">
      <div class="scala-code">
        <div class="code-element">
          <div class="bar-code"><span>remove-boilerplate.scala</span></div>
          <pre><code>enum Employee derives Codec:
case Basic(id: Id, name: Name)
case WithAge(id: Id, name: Name, age: Age)

val json = employee.encode</code></pre>
        </div>
      </div>
      <div class="scala-text">
        <p>Declare data structures without boilerplate</p>
      </div>
    </div>
  </div>
</div>
