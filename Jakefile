var JAKE = require("../jake/lib/jake");
var OS = require("os");

var util = require('util');
var fs = require('fs');
var path = require('path');
var child_process = require('child_process');

var VERSION = "1.0";

var BUILD_PATH = path.resolve("Build");
var SERVER_BUILD_PATH = path.join(BUILD_PATH, "Server");
var CLIENT_BUILD_PATH = path.join(SERVER_BUILD_PATH, "static");

var DEPLOY_GIT_REMOTE = "git@heroku.com:githubissues.git";

OS.enquote = function (word) {
    return "'" + String(word).replace(/'/g, "'\"'\"'") + "'";
};

exports.copyTree = function(source, target, basePath) {
    var sourcePath = (source = path.resolve(basePath, source));
    var targetPath = (target = path.resolve(basePath, target));
    if (path.existsSync(targetPath))
        throw new Error("file exists: " + targetPath);
	if (!path.existsSync(sourcePath))
		throw new Error("file does not exist: " + sourcePath);
    if (fs.statSync(sourcePath).isDirectory()) {
        fs.mkdirSync(targetPath);
        fs.readdirSync(sourcePath).forEach(function(name){
            exports.copyTree(source, target, path.join(basePath, name));
        });
    } else {
        fs.writeFileSync(targetPath, fs.readFileSync(sourcePath));
    }
};

JAKE.task("deploy", ["build"], function(cb) {
    var sha = getGitSHA();

    child_process.exec("cd "+OS.enquote(SERVER_BUILD_PATH)+" " + buildCommandString([
        ["git", "pull"],
        ["git", "add", "."],
        ["git", "commit", "-a", "-m", sha+" built on "+(new Date())],
        ["git", "push", "heroku", "master"]
    ]), cb);
});

JAKE.task("build", ["checkout"], function(cb) {
    // server
    child_process.exec("cp -r Server/* " + OS.enquote(path.resolve(SERVER_BUILD_PATH)), function(){
	    // client
	    child_process.exec("cd Client && jake deploy", function(){
		    if (path.exists(CLIENT_BUILD_PATH) && fs.statSync(CLIENT_BUILD_PATH).isDirectory()){
		        fs.rmdirSync(CLIENT_BUILD_PATH);
			}

		    exports.copyTree(path.join("Client", "Build", "Deployment", "Issues"), CLIENT_BUILD_PATH);
			cb();
		});
	});
});

JAKE.task("checkout", function(cb) {
    if (!(path.exists(SERVER_BUILD_PATH) && fs.statSync(SERVER_BUILD_PATH).isDirectory())) {
        fs.mkdir(path.dirname(SERVER_BUILD_PATH), 511, function(){
	        child_process.exec(["git", "clone", DEPLOY_GIT_REMOTE, SERVER_BUILD_PATH, "-o", "heroku"], function(){
				cb();
			});
		});
    }
});

JAKE.task("zip", function() {
    var zipName = "Issues-"+VERSION+".zip";
    if (BUILD_PATH.join(zipName).isFile())
        BUILD_PATH.join(zipName).remove();

    child_process.exec("cd Client && jake desktop", function(){
	    child_process.exec("cd Client/Build/Desktop/Issues && zip -r ../../../Build/"+zipName+" Issues.app", cb);
	});
});

function getGitSHA(directory) {
    var p = fs.readfile((directory ? "cd "+OS.enquote(directory)+" && " : "") + "git rev-parse --verify HEAD");
    var sha = p.stdout.read().trim();
    p.stdin.close();
    p.stdout.close();
    p.stderr.close();
    return sha;
}

function buildCommandString(commands) {
    return commands.map(function(command) {
        return command.map(OS.enquote).join(" ");
    }).join(" && ");
}
