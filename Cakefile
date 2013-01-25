fs            = require 'fs'
{spawn, exec} = require 'child_process'
path          = require 'path'
findit        = require 'findit'
util          = require 'util'
glob          = require 'glob'
{print}       = require 'sys'


SRCDIR="./src"
TGTDIR="./lib"
 
find_all = (srcdir, src_prefix) ->
  
mkr = (r) -> new RegExp r.replace /\./, '\\.'

twoDigits = (val) ->
  if val < 10 then return "0#{val}" 
  return val

#######################################

build = (src_extension, targ_extension, command, options) ->
    findit.find(SRCDIR).on 'file', (source, stats) ->
        if path.extname(source) == src_extension
            source_path = path.dirname(source)
            source_file = path.basename(source)
            target_path = source_path.replace(mkr('^' + SRCDIR), TGTDIR)
            target      = path.join(target_path, source_file)
            target_file = path.basename(target)
            fs.mkdir target_path, '0755', (err) ->
                if err and err.code != 'EEXIST'
                    throw err
            cmd = command.replace(/\$source/, source)
                .replace(/\$target_path/, target_path)
                .replace(/\$source_path/, source_path)
                .replace(/\$target_file/, target_file)
                .replace(/\$source_file/, source_file)
                .replace(/\$target/, target)
            exec cmd, (error, stderr, stdout) ->
                if error
                    throw source + ' ' + error
                if options.verbose
                    console.log(stdout)
                if stderr
                    console.log(stderr)

task 'docs', 'generate api documentation', (options) ->
  #coffeedoc = spawn 'coffeedoc' , []
  exec "coffeedoc --output ./doc/api --parser commonjs ./src/"
  
task 'watch', 'Watch src/ for changes',(options) ->
  coffee = spawn 'coffee', ['-w', '-c', '-o', TGTDIR, SRCDIR]
  coffee.stderr.on 'data', (data) ->
    process.stderr.write data.toString()
  coffee.stdout.on 'data', (data) ->
    print data.toString()

task 'serve', 'minimal web server for testing', (options) ->
  connect = require('connect')
  servePath = path.resolve('.')
  server = connect.createServer connect.static(servePath)
  server.listen(8090)

task 'serveWatch', 'serve the files and run the watch task', (options) ->
  invoke('serve')
  invoke('watch')

      ###  
task 'build', 'build all the components', (options) ->
  build '.coffee', '.js', 'coffee --compile --lint -o $target_path $source', options
  
task 'release', 'build, minify , prep for release' , (options) ->
  c = {baseUrl: 'app'
      ,name:    'main.max'
      ,out:     'build/backbone.dropbox.js'
      ,optimize:'uglify'}
  requirejs.optimize(c, console.log)
