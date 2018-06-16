# 🤖 Discord.js example bot
Example Discord Bot written in Discord.js 

## :computer: Use
If you don't have git or node, you can install they here [Git](https://git-scm.com/downloads "Git") [Node.js](https://nodejs.org/en/download/ "Node.js") 

    # Clone this repository
    git clone https://github.com/KrystianJonca/discord.js-example-bot
    # Go into the repository
    cd discord.js-example-bot
    # Install dependencies
    npm install
    # Run the app
    npm start

## :file_folder: bot.js
```javascript
// Import require modules
const Discord = require('discord.js');

// Require config file with prefix,token
const config = require('./config.json');

// Create our bot
const bot = new Discord.Client();

// Get our prefix from config file
let prefix = config.prefix;

// Create ready event
bot.on('ready',() =>{
    // This function will run when bot starts
    console.log(`I am ready to use - ${bot.user.username}`);
    bot.user.setUsername(config.username) // Set bot username(optional)

    let activities = [
        `with ${bot.users.size} users!`,
        `on ${bot.channels.size} channels`, 
        `for ${(bot.uptime/3600000).toFixed(2)} h!`, 
        `with ${(bot.ping).toFixed(0)} ping!`
    ]; // Our activities array
    let activityNumber = Math.floor(Math.random()*activities.length); // Generate random activity number
    bot.user.setActivity(activities[activityNumber]); // Set random bot activity from activities array

    bot.setInterval(() => { // We have to update bot activities that is why I created this interval
        activities = [
            `with ${bot.users.size} users!`,
            `on ${bot.channels.size} channels`, 
            `for ${(bot.uptime/3600000).toFixed(2)} h!`, 
            `with ${(bot.ping).toFixed(0)} ping!`
        ]; // Update our activities array
        activityNumber = Math.floor(Math.random()*activities.length); // update random activity number
        bot.user.setActivity(activities[activityNumber]); // and finally update bot activity
    },600000) // 600000ms - 10 min
})

// Create message event listener
bot.on('message',async message =>{
    // This function will run on every single message received, from any channel or DM

    if(message.author.bot) return; // Ignore other bots
    if (message.channel.type === "dm") return; // Ignore DM messages

    // Helpers
    let sender = message.author;    
    let content = message.content;
    let contentArray = content.split(" ");
    let command = contentArray[0];
    let cmd = command.slice(prefix.length).toLowerCase()
    let args = contentArray.slice(1);
    
    if (!command.startsWith(prefix)) return; // Ignore messages which doesn't start with prefix

    // Let's create some commands!
    
    switch (cmd) {
        case "ping":
            message.reply(`Pong! (${bot.ping.toFixed(0)})`) // Send message with user mention to the same channel. If you want reply without user mention use message.channel.send()
            break;
        case "avatar":
            let msg = await message.channel.send("Loading avatar..."); // Send temporary message

            await message.channel.send({files: [
                {
                    attachment: message.author.displayAvatarURL,
                    name: "avatar.png"
                }
            ]}); // Send message with user avatar file
            msg.delete(); // and then delete temporary message
            break;
        case "roll":
            let roll = Math.floor(Math.random()*6)+1; // Generate random number between 1 and 6

            message.reply(`You rolled a ${roll}`); // Send message with rolled number
            break;
        case "userinfo":
                let target = message.mentions.users.first() || message.guild.members.get(args[0]) || message.author; // Get user target(user mention or if it doesn't exist message author)

                let embed = new Discord.RichEmbed()
                    .setAuthor(target.username)
                    .setDescription("User's info")
                    .setColor("#22A7F0")
                    .setThumbnail(target.displayAvatarURL)
                    
                    .addField("Full Username", `${target.username} #${target.discriminator}`)
                    .addField("ID", target.id)
                    .addField("Created at", target.createdAt);// Create discord embed
                message.channel.send(embed); // Send discord embed
                break;
        default:
            message.reply('This command does not exist!').then(msg => {msg.delete(5000)}); // If the command does not exist, send an informational message and delete it after 5 sec
            break;
    }
})

// Additions events
bot.on("guildCreate", guild => {
    console.log(`Joined a new guild: ${guild.name}(id: ${guild.id}). This guild has ${guild.memberCount} members!`);
})
bot.on("guildDelete", guild => {
    console.log(`I have been removed from: ${guild.name} (id: ${guild.id})`);
})
bot.on("debug", info => {
    console.log(info); // This will output your token when the bot starts
});
bot.on("disconnect", event => {
    console.log(event);
});
bot.on("reconnecting",() => {
    console.log("Reconnecting");
});

// Log our bot in
bot.login(config.token);


```