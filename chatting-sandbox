// ======<server.js>======================================================
const querystring = require('querystring');
const http = require('http');
const EventEmitter = require('events');
var ee = new EventEmitter();

function INFO(tag, message) {
        console.log(`[INFO] ${tag} : ${message}`);
}

http.createServer(function (req, res) {
        var datas = "";
        var dataTimeout = function() {
                datas = querystring.parse(datas);
                if (datas.data === 'listen') {
                    INFO(`Listening`, `${datas.cid_from} is listening`);
                    ee.on(datas.cid_from, function(data) {
                        res.write(querystring.stringify(data));
                    });
                } else {
                    res.end();
                }
        }

        req.on('data', data => {
                datas += data;
                clearTimeout(dataTimeout);
                setTimeout(dataTimeout, 1000);
        });

        req.on('end', () => {
                clearTimeout(dataTimeout);
                datas = querystring.parse(datas);
                ee.emit(datas.cid_to, datas);
                INFO(`Client ${datas.cid_from}`, datas.data);
                res.end();
        });
}).listen(3030);

// ======<client.js>======================================================
// r e q u i r e
const http = require('http');
const qstr = require('querystring');

// CONSTANT DECLARATION
const p = {};

p.inp = process.stdin;
p.out = process.stdout;
p.data = "";
p.datac = 0;
p.client_id = 0;
p.client_to = 0;
p.listen = false;

p.host = '127.0.0.1';
p.port = '3030';
p.path = '';

// dependencies functors
function INFO(tag, message) {
        console.log(`[INFO] ${tag} : ${message}`);
}

function listen() {
        var pdata = qstr.stringify({
                cid_from : p.client_id,
                data : 'listen'
        });

        var poptions = {
                host : p.host,
                port : p.port,
                method : 'POST',
                header : {
                        'Content-Type' : 'text/plain',
                        'Content-Length' : Buffer.byteLength(pdata)
                }
        }

        var preq = http.request(poptions, function(res) {
                res.setEncoding("ascii");
                res.on('data', data => {
                        data = qstr.parse(data);
                        INFO(`from ${data.cid_from}`, data.data);
                });
        });

        preq.write(pdata);
}

function chat(message) {
        var pdata = qstr.stringify({
                cid_from : p.client_id,
                cid_to : p.client_to,
                data : message 
        });

        var poptions = {
                host : p.host,
                port : p.port,
                method : 'POST',
                header : {
                        'Content-Type' : 'text/plain',
                        'Content-Length' : Buffer.byteLength(pdata)
                }
        }

        INFO(`Sent to ${p.client_to}`, message);

        var preq = http.request(poptions, function(res) {
                var datas = "";
                res.setEncoding("ascii");
                res.on('data',  data => datas += data);
                res.on('end', () => {
                  if (datas) 
                      INFO('RESPONSE', qstr.parse(datas).data)
                });
        });

        preq.write(pdata);
        preq.end();
}

// input_process
INFO("WARNING", "[:sc your_client_id] to setup your client id");
INFO("WARNING", "You can not send anything to server unless you set it");
INFO("CHAT", "--------entering chat client---------");

p.inp.resume();
p.inp.setEncoding('ascii');

p.inp.on('data', function (data) {
        data = data.split("\n")[0];

        if (data === ":q") {
                p.inp.pause();
                return;
        }

        if (/^:sc \d+/.test(data)) {
                p.client_id = data.match(/\d+/)[0];
                if (p.client_id == 0)
                        return;

                INFO("client_id", `changed to : ${p.client_id}`);
                listen();
                return;
        }

        if (/^:st \d+/.test(data)) {
                p.client_to = data.match(/\d+/)[0];
                INFO("Message to", p.client_to);
                return;                
        }

        if (p.client_id != 0) 
                chat(data);
});
