<html>
<head>
<title>Butcher Block Extension Builder</title>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="jszip.min.js"></script>
<script src="FileSaver.min.js"></script>
<script>
function handleClick() {
  $.getJSON('manifest_template.json', function(manifest) {
    $.getJSON('rule_template.json', function(rule) {
      var ext_name = document.getElementById('ext_name').value;
      var meet_ids = document.getElementById('meet_ids').value.split('\n');
      manifest.name = ext_name;
      var nowd = new Date();
      var year = nowd.getUTCFullYear().toString();
      var month = (nowd.getUTCMonth() + 1).toString().padStart(2, "0");
      var dom = nowd.getUTCDate().toString().padStart(2, "0");
      var hour = nowd.getUTCHours().toString().padStart(2, "0");
      var minutes = nowd.getUTCMinutes().toString().padStart(2, "0");
      var seconds = nowd.getUTCSeconds().toString().padStart(2, "0");
      var ver_str = `${year}.${month}${dom}.${hour}.${minutes}${seconds}`;
      manifest.version = ver_str;
      var rules = [];
      var this_rule;
      var meet_id;
      var meet_regex;
      var this_rule;
      for(var i = 0;i < meet_ids.length;i++) {
        meet_id = meet_ids[i];
        meet_id = meet_id.replace(/-/g, '');
        meet_id = meet_id.toLowerCase();
        if (meet_id.length != 10) {
          continue;
        }
        meet_regex = `(?i)${meet_id.substring(0, 3)}[-]?${meet_id.substring(3, 7)}[-]?${meet_id.substring(7, 10)}`;
        this_rule = $.extend( true, {}, rule);
        this_rule["condition"]["regexFilter"] = meet_regex;
        this_rule["id"] = i + 1;
        rules.push(this_rule);
      }
      var zip = new JSZip();
      // split into rulesets of 1k each
      var i, j, ruleset, filename, temparray,chunk = 1000;
      var n = 0
      for (i=0,j=rules.length; i<j; i+=chunk) {
        n = n + 1;
        ruleset = rules.slice(i, i+chunk);
        filename = `rules${n}.json`
        zip.file(filename, JSON.stringify(ruleset, null, 2));
        manifest["declarative_net_request"]["rule_resources"].push({"id": `ruleset_${n}`, "enabled": true, "path": filename});
      }
      zip.file("manifest.json", JSON.stringify(manifest, null, 2));
      zip.generateAsync({type:"blob"}).then(function(content) {
        saveAs(content, "example.zip");
      });
    });
  });
}

</script>
</head>
<body>
  <h2>Meet Blocking Extension Generator</h2>
<form name="exdetails" method="post" onSubmit="handleClick(); return false">
        Name your extension: <input type="text" id="ext_name" name="ext_name"><br>
        List Meet IDs to be blocked:<br>
        <textarea id="meet_ids" name="meet_ids" rows="25" cols="20"></textarea><br>
        <br>
        <input name="Submit"  type="submit" value="Generate Extension" />
</form>

</body>
