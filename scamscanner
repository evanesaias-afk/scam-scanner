const {
  Client,
  GatewayIntentBits,
  SlashCommandBuilder,
  REST,
  Routes,
  EmbedBuilder
} = require('discord.js');

// ===============================
// CONFIG FROM RENDER ENVIRONMENT
// ===============================
const TOKEN = process.env.DISCORD_TOKEN;
const CLIENT_ID = process.env.CLIENT_ID;
const GUILD_ID = process.env.GUILD_ID;
const SCAM_CHANNEL_ID = process.env.SCAM_CHANNEL_ID;

// Your scammer role ID
const SCAM_ROLE_ID = '1404359573401370705';

if (!TOKEN || !CLIENT_ID || !GUILD_ID || !SCAM_CHANNEL_ID || !SCAM_ROLE_ID) {
  console.error('Missing environment variables.');
  console.error('Required: DISCORD_TOKEN, CLIENT_ID, GUILD_ID, SCAM_CHANNEL_ID');
  process.exit(1);
}

// ===============================
// REGISTER SLASH COMMAND
// ===============================
const commands = [
  new SlashCommandBuilder()
    .setName('scammer')
    .setDescription('Report a scammer')
    .addUserOption(option =>
      option
        .setName('user')
        .setDescription('The scammer')
        .setRequired(true)
    )
    .addStringOption(option =>
      option
        .setName('reason')
        .setDescription('Reason for report')
        .setRequired(true)
    )
].map(command => command.toJSON());

const rest = new REST({ version: '10' }).setToken(TOKEN);

async function registerCommands() {
  try {
    console.log('Deploying slash command...');

    await rest.put(
      Routes.applicationGuildCommands(CLIENT_ID, GUILD_ID),
      { body: commands }
    );

    console.log('Slash command registered.');
  } catch (error) {
    console.error('Failed to register slash command:');
    console.error(error);
  }
}

// ===============================
// CREATE BOT
// ===============================
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMembers
  ]
});

client.once('ready', () => {
  console.log(`Logged in as ${client.user.tag}`);
});

// ===============================
// COMMAND HANDLER
// ===============================
client.on('interactionCreate', async interaction => {
  if (!interaction.isChatInputCommand()) return;
  if (interaction.commandName !== 'scammer') return;

  const user = interaction.options.getUser('user');
  const reason = interaction.options.getString('reason');
  const guild = interaction.guild;

  const channel = await guild.channels.fetch(SCAM_CHANNEL_ID).catch(() => null);

  if (!channel) {
    return interaction.reply({
      content: 'Scam channel not found. Check SCAM_CHANNEL_ID.',
      ephemeral: true
    });
  }

  const member = await guild.members.fetch(user.id).catch(() => null);

  if (!member) {
    return interaction.reply({
      content: 'User not found in this server.',
      ephemeral: true
    });
  }

  try {
    await member.roles.add(SCAM_ROLE_ID);
  } catch (error) {
    console.error('Failed to give role:');
    console.error(error);

    return interaction.reply({
      content: 'I could not give the role. Make sure my bot role is above the scammer role and I have Manage Roles permission.',
      ephemeral: true
    });
  }

  const alertMessage =
    `🚨 Scammer Alert\n\n` +
    `User: ${user.tag} (${user.id})\n` +
    `Mention: <@${user.id}>\n` +
    `Reason: ${reason}`;

  const embed = new EmbedBuilder()
    .setColor(0x8B0000)
    .setTitle('🚨 Scammer Alert')
    .setDescription(
      `User: ${user.tag} (${user.id})\n` +
      `Mention: <@${user.id}>\n` +
      `Reason: ${reason}`
    )
    .setImage('https://cdn.discordapp.com/attachments/1404360337079271504/1404617327151943812/scammer.gif')
    .setTimestamp();

  await channel.send({
    content: `<@${user.id}> has been marked as a scammer.`,
    embeds: [embed]
  });

  let dmStatus = 'DM sent.';

  try {
    await user.send({
      content: alertMessage,
      embeds: [embed]
    });
  } catch (error) {
    console.error('Failed to send DM:');
    console.error(error);
    dmStatus = 'DM failed. Their DMs might be closed.';
  }

  await interaction.reply({
    content: `Scammer reported. Role added. ${dmStatus}`,
    ephemeral: true
  });
});

// ===============================
// START BOT
// ===============================
(async () => {
  await registerCommands();
  await client.login(TOKEN);
})();
