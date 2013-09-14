template: default
---
# Archives

<ul class="archives">
: for $blog.entries() -> $entry {
<li><time><: $entry.published_at.strftime('%Y-%m-%d') :></time> <a href="<: $entry.site_path() | uri_for :>"><: $entry.title :></a> 
  <a href="http://b.hatena.ne.jp/entry/<: $entry.site_path | uri_for :>"><img src="http://b.hatena.ne.jp/entry/image/small/<: $entry.site_path | uri_for :>"></a>
</li>
: }
</ul>
