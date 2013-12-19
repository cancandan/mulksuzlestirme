mulksuzlestirme
===============

Bu proje [mulksuzlestirme.org](http://mulksuzlestirme.org) projesindeki ag datasi uzerinde bir graphdb kullanarak sorgulama yapabilme denemesidir.

Kullanilan graph veritabani [HeliosJS](http://entrendipity.github.io/helios.js/) browserda in-memory calisan bir veri tabani. Gremlin benzeri bir sorgulama dili kullaniyor, nasil sorgulama yapildigini anlamak icin lutfen sitesine bakin,

Bu denemede acilan sayfadaki karede (javascript console) helios komutu calistirabiliyoruz, ornegin
```
graphdb.v(4).id()
```

girersek id si 4 olan vertex leri getiriyor ve ```var idler``` degiskenine atiyor. Kullanilan javascript konsolunun kodunda helios u calistirmak icin biraz degisiklik yaptim, altta detayi var.
[http://josscrowcroft.github.io/javascript-sandbox-console/](http://josscrowcroft.github.io/javascript-sandbox-console/) 

bundan sonra ```goster()``` fonksiyonu cagrilirsa (goser() yazip entera basarak) o nodelari haritada gosteriyor.


Kod eklentileri/degisiklikleri
==============================


Javascript console ile ilgili
-----------------------------

graphdb ye query yaptiktan sonra sonuclar 
```var idler``` 
degiskeninde durmus oluyor bunu yapmak icin sandbox-console.js de evaluate fonksiyonunu heliosun calisma sekline gore degistirdim, sandbox-console.js deki ilgili kod 

```
var myself=this;
			function changeItem(r) {
				item.result=r;
				idler=r;
				return myself.addHistory(item);
			}
			
			// Evaluate the command and store the eval result, adding some basic classes for syntax-highlighting
			try {
				//item.result = this.get('iframe') ? this.iframeEval(command) : eval.call(window, command);
				//eval(command).then(function(result){item.result=JSON.stringify(result)});
				if (command.indexOf('graph')==-1) {
					item.result = this.get('iframe') ? this.iframeEval(command) : eval.call(window, command);
					this.addHistory(item);
				} else {
				eval(command).then(
					function(result) {
						changeItem(result);
					});
				}
				...
```

Javascript Console u gosteren kod
```
<div id="sandbox">sandbox yukleniyor...</div>

<!-- sandboxla ilgili kod -->
<!-- The sandbox template -->
<script type="text/template" id="tplSandbox">
  <pre class="output"></pre>
  <div class="input">
    <textarea rows="1" placeholder="<%= placeholder %>"></textarea>
  </div>
  <a href='http://entrendipity.github.io/helios.js/' target="_blank">Komutlar icin HeliosJS sayfasi http://entrendipity.github.io/helios.js/</a>
</script>


<!-- The command/result template (NB whitespace/line breaks matter inside <pre> tag): -->
<script type="text/template" id="tplCommand"><% if (! _hidden) { %><span class="command"><%= command %></span>
<span class="prefix"><%= this.resultPrefix %></span><span class="<%= _class %>"><%= result %></span>
<% } %></script>


<script src="js/libs/underscore.min.js"></script>
<script src="js/libs/backbone.min.js"></script>
<script src="js/libs/backbone-localstorage.min.js"></script><!-- opt -->
<script src="js/libs/jquery.min.js"></script>

<script src="js/sandbox-console.js"></script>
<script type="text/javascript">
  jQuery(document).ready(function($) {
    // Create the sandbox:
    window.sandbox = new Sandbox.View({
      el : $('#sandbox'),
      model : new Sandbox.Model(),
      placeholder : "yardim icin :yardim, Ornek graphdb.v(4).id()",
      helpText : "idler degiskenini graphdb query yaparak doldurun\n\n mesela graphdb.v(4,5).id()\n\n yerlerini grafikte gormek icin, goster()\n fonksiyonunu cagirin\n\n baska bir ornek:\n idler=[3,8,10];goster() \n\n yukardaki alani silmek icin :sil"
    });
  });
</script>
```


Heliosun verileri okumasi ile ilgili
------------------------------------
```
<script>
  var rawjson;
  var graphdb;
  var g;
  var data;
  var idler;

  function goster() {
      resetGraph();    
      highlightSomeNodes(idler);
  }

  <!-- Json datasini Helios un anladigi GraphJSON a donusturme ve yukleme -->
  $($.get(
  'http://www.corsproxy.com/mulksuzlestirme.org/yonetimkurulu-neo.json?callback=hey',
    function(response) {
      rawjson=response;
      var vertices=[]
      var edges=[]

      rawjson.nodes.forEach(function(entry) {
        v={"name":entry.nodeName, "_id": entry.id, "_type": "vertex"};
        vertices.push(v)
      });

      rawjson.links.forEach(function(entry) {
        e={"_type": "edge", "_outV": entry.target, "_inV": entry.source, "_label": entry.type};
        edges.push(e)
      });


      data = {
          "graph": {
              "mode":"NORMAL",
              "vertices":vertices, "edges":edges}};
      graphdb = new Helios.GraphDatabase({heliosDBPath:'lib/heliosDB.js'});
      g=graphdb.loadGraphSON(data);   

    }));

</script>
```



