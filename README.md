# discord
a crappy overview on how to make discord webpack scripts (not betterdiscord scripts but rather actual reverse engineering stuff)

###### idk shit about webpack all of this is from personal experience

## Before anything (if you are on the desktop client):

go to `%appdata%\discord\settings.json` and add
```js
"DANGEROUS_ENABLE_DEVTOOLS_ONLY_ENABLE_IF_YOU_KNOW_WHAT_YOURE_DOING": true
```
to the end of the object, it allows the developer console to be opened with ctrl+shift+i

## actual "tutorial"
Alright so:
Discord runs mainly on Electron, React and webpack

*almost* every single discord feature is accessible from the
```js
window.webpackChunkdiscord_app
```
array, which contains every single webpack functions and stuff that discord relies on

"but hey dude how in the world am i supposed to actually abuse it its jsut a gigantic bunch of \[\[1231313],{123123213:fafasfasdfsf},()=>{}]"

simple:
```js
window.webpackChunkdiscord_app.push(
  [
    [Math.random()],
    {},
    (req)=>{}
  ]
)
```
when pushing this array into the webpackChunk, the `(req)=>{}` function will instantly execute

the `req` thing is a function that you can pass a number to it, and it returns an object whose contents may vary a lot

**however:**

doing `req.c` returns an object with every single function-holding object loaded (which is where all the functions are)
it looks something like:
```js
{
  NUMERIC_ID:{
    exports: {
      //there may be other object with a random 1-3 letter name such as Z, Plq, ZP or whatever
      //thats where the functions actually are
    }
    id: NUMERIC_ID, 
    //note: passing the id to the `req` function returns the `exports` object
    //personally, since discord updates a lot i dont rely on using this number because it can update
    //i just search though the object instead
    loaded: true
  }
}
```

so, for example:
```js
window.webpackChunkdiscord_app.push([[Math.random()],{},(req)=>{
  for(const thing of req.c){
    if(thing.exports.Z.getCurrentUser){
      return thing.exports.Z
    }
  }
}])

/*
{
  commands: {...},
  events: {...},
  getCurrentUser: f(), //this one is interesting
  getJoi: f(),
  onConnect: f(e),
  onDisconnect: f(e,t),
  sockets: Set(0) {size: 0},
  subscriptions: []
}

and getCurrentUser returns all the information for your currently logged in user (except for token)
*/

// and for the token:

window.webpackChunkdiscord_app.push([[Math.random()],{},(req)=>{
  for(const thing of req.c){
    if(thing.exports.Z.getToken){
      return thing.exports.Z.getToken()
    }
  }
}])

//returns the token :)
```

There are many snippets out there, i will include some tomorrow, but for now, if you want to further analyze the webpack script source code easily (the discord app one is minified), use [Discord-Datamining](https://github.com/Discord-Datamining/Discord-Datamining)

# some snippets i have
1. clyde saying the funny
```js
let amongus;
window.webpackChunkdiscord_app.push([[Math.random()],{},(a)=>{amongus=a}])
Object.values(amongus.c).find(x=>x.exports?.Z?.sendBotMessage).exports.Z.sendBotMessage(window.location.pathname.slice(10).split('/')[1],'mongus balls')
```
![clyd](https://host.paderos-neko.store/raw/Discord_8Ngm4uNAwZ_iOUqWTjpFREopgV.png)

2. (somewhat broken idk why) tells you if you have embed failure (no EMBED_LINKS permission)
```js
const pair = window.location.pathname.slice(10).split('/')
let amongus;
window.webpackChunkdiscord_app.push([[Math.random()],{},(a)=>{amongus=a}])
let uid = Object.values(amongus.c).find(x=>x.exports?.Z?.getCurrentUser).exports.Z.getCurrentUser().id
let member = Object.values(amongus.c).find(x=>x.exports?.ZP?.__proto__?.getSelfMember).exports.ZP.__proto__.getMembers(pair[0]).find(x=>x.userId==uid)
let guild = Object.values(amongus.c).find(x=>x.exports?.Z?.getGuilds).exports.Z.__proto__.getGuild(pair[0])
let channel = Object.values(amongus.c).find(x=>x.exports?.Z?.getChannel).exports.Z.getChannel(pair[1])
let has = Object.values(amongus.c).find(x=>x.exports?.Z?.asUintN).exports.Z.has
const perms = Object.values(amongus.c).find(x=>x.exports?.Plq).exports.Plq
let piss = false
for(const role of member.roles){
    if(has(guild.roles[role].permissions,perms.ADMINISTRATOR)){
        piss = true
    } else if(has(guild.roles[role].permissions,perms.EMBED_LINKS)){
        if(Object.keys(channel.permissionOverwrites).includes(role) && !has(channel.permissionOverwrites[role].deny,perms.EMBED_LINKS)) piss = true
    } else if(Object.keys(channel.permissionOverwrites).includes(role) && has(channel.permissionOverwrites[role].allow,perms.EMBED_LINKS)) piss = true
}
console.log(piss ? 'you dont have embed failure big W' : 'you have embed failure big LLLLLLLL')
```
