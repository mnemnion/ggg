This guide, jacked from the Instaparse reference, may prove useful:

<table>
<tr><th>String syntax</th><th>Combinator</th><th>Mnemonic</th></tr>
<tr><td>Epsilon</td><td>Epsilon</td><td>Epsilon</td></tr>
<tr><td>A | B | C</td><td>(alt A B C)</td><td>Alternation</td></tr>
<tr><td>A B C</td><td>(cat A B C)</td><td>Concatenation</td></tr>
<tr><td>A?</td><td>(opt A)</td><td>Optional</td></tr>
<tr><td>A+</td><td>(plus A)</td><td>Plus</td></tr>
<tr><td>A*</td><td>(star A)</td><td>Star</td></tr>
<tr><td>A / B / C</td><td>(ord A B C)</td><td>Ordered Choice</td></tr>
<tr><td>&A</td><td>(look A)</td><td>Lookahead</td></tr>
<tr><td>!A</td><td>(neg A)</td><td>Negative lookahead</td></tr>
<tr><td>&lt;A&gt;</td><td>(hide A)</td><td>Hide</td></tr>
<tr><td>"string"</td><td>(string "string")</td><td>String</td></tr>
<tr><td>#"regexp"</td><td>(regexp "regexp")</td><td>Regular Expression</td></tr>
<tr><td>A non-terminal</td><td>(nt :non-terminal)</td><td>Non-terminal</td></tr>
<tr><td>&lt;S&gt; = ...</td><td>{:S (hide-tag ...)}</td><td>Hide tag</td></tr>
</table>

