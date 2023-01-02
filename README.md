
process.on('unhandledRejection', (reason, p) => {
  console.log(' [ ANTICLASH ] | SCRIPT REJEITADO');
  console.log(reason, p);
});

process.on("uncaughtException", (err, origin) => {
  console.log(' [ ANTICLASH] | CATCH ERROR');
  console.log(err, origin);
})

process.on('uncaughtExceptionMonitor', (err, origin) => {
  console.log(' [ ANTICLASH ] | BLOQUEADO');
  console.log(err, origin);
});

process.on('multipleResolves', (type, promise, reason) => {
  console.log(' [ ANTICLASH ] | VÁRIOS ERROS');
  console.log(type, promise, reason);
});

const Discord = require("discord.js")

const config = require("./config.json")

const { parseArgs } = require("util");
const { QuickDB } = require("quick.db");
const db = new QuickDB()

const client = new Discord.Client({
  intents: [
    959,
    Discord.IntentsBitField.Flags.MessageContent
  ]
});

module.exports = client

client.on('interactionCreate', (interaction) => {

  if (interaction.type === Discord.InteractionType.ApplicationCommand) {

    const cmd = client.slashCommands.get(interaction.commandName);

    if (!cmd) return interaction.reply(`Error`);

    interaction["member"] = interaction.guild.members.cache.get(interaction.user.id);

    cmd.run(client, interaction)

  }
})

client.on('ready', () => {
  console.log(`🔥 Estou online em ${client.user.username}!`)
})


client.slashCommands = new Discord.Collection()

require('./handler')(client)

client.login(config.token)

//////// [ WHITE LIST ] //////

client.on('ready', () => {

  let servidor = client.guilds.cache.get("1039626431451373639") // Coloque o ID do servidor.
  let canal = servidor.channels.cache.get("1039626431937921104") // Coloque o ID do canal de texto

  let botao = new Discord.ButtonBuilder()
    .setCustomId('wl')
    .setLabel('ABRIR WHITELIST')
    .setEmoji("📄")
    .setStyle(Discord.ButtonStyle.Secondary);

  let botao_wl = new Discord.ActionRowBuilder().addComponents(botao);

  let embed = new Discord.EmbedBuilder()
    .setColor("FF0000")
    .setAuthor({ name: servidor.name, iconURL: servidor.iconURL({ dynamic: true }) })
    .setDescription(`Clique no botão abaixo para iniciar sua whitelist!`)
    .setFooter({ text: `⏰ Você terá 1 minuto para responder cada pergunta.` });

  canal.send({ embeds: [embed], components: [botao_wl] })
  // Após dar "node .", adicione duas barras (//) no início da linha de cima, para poder cancelar o envio da mensagem toda vez que o bot ligar.

});

client.on('interactionCreate', async interaction => {

  if (!interaction.isButton()) return;

  let categoria = interaction.guild.channels.cache.get("1039626432206348299") // Coloque o ID da categoria
  let cor_embeds = "FF0000" // Coloque o código da cor para as embeds

  if (!cor_embeds) cor_embeds = "Random";
  if (interaction.isButton()) {

    if (interaction.customId.startsWith('wl')) {

      /*if (await db.get(`user_wl_${interaction.user.id}`) === true) return interaction.reply(`${interaction.user} Você já fez sua WhiteList, portanto não poderá fazer novamente!`).then(() => {
        setTimeout(() => {
          interaction.deleteReply()
        }, 5000)
      });*/

      let canal = interaction.guild.channels.cache.find(c => c.name === `📝┃wl-${interaction.user.id}`);

      if (canal) {
        interaction.reply(`${interaction.user} Você já possui uma WhiteList aberta em ${canal}.`).then(() => {
          setTimeout(() => {
            interaction.deleteReply()
          }, 5000)
        })
      } else if (!canal) {

        interaction.guild.channels.create({
          name: `📝┃wl-${interaction.user.id}`,
          type: Discord.ChannelType.GuildText,
          parent: categoria.id,

          permissionOverwrites: [
            {
              id: interaction.guild.id,
              deny: [
                Discord.PermissionFlagsBits.SendMessages,
                Discord.PermissionFlagsBits.ViewChannel,
                Discord.PermissionFlagsBits.ReadMessageHistory,
                Discord.PermissionFlagsBits.AttachFiles
              ]
            },
            {
              id: interaction.user.id,
              allow: [
                Discord.PermissionFlagsBits.SendMessages,
                Discord.PermissionFlagsBits.ViewChannel,
                Discord.PermissionFlagsBits.ReadMessageHistory,
                Discord.PermissionFlagsBits.AttachFiles
              ]
            }
          ]
        }).then(async (canal) => {

          interaction.reply(`Olá ${interaction.user}, a sua WhiteList foi aberta em ${canal}.`).then(m => {

            setTimeout(() => {
              interaction.deleteReply()
            }, 5000)

          })

          let embed_1 = new Discord.EmbedBuilder()
          .setColor(cor_embeds)
          .setAuthor({ name: interaction.guild.name, iconURL: interaction.guild.iconURL({ dynamic: true }) })
          .setTitle(`__Pergunta [01]__`)
          .setDescription(`> **Qual é o seu nome?**\n\n\\⏰ Você possui 5 minutos para completar tudo.`);
          /*
            .setColor(cor_embeds)
            .setAuthor({ name: interaction.guild.name, iconURL: interaction.guild.iconURL({ dynamic: true }) })
            .setTitle(`WhiteList Iniciada!`)
            .setDescription(`\\🔔 ${interaction.user} Aqui está sua WhiteList.\n\n\➡ Aguarde 5 segundos para iniciar sua WhiteList!\n\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);
          */

          canal.send({ content: `${interaction.user}`, embeds: [embed_1] }).then(async msg => {
            msg.pin();
            await db.set(`wl_${interaction.user.id}`, msg.id);
          })
        })

      }
    }
  }
});

client.on('messageCreate', async (message) => {

  if (message.channel.type === Discord.ChannelType.DM) return;


  let chat_logs = client.channels.cache.get("1039626431937921101"); // Coloque o ID do chat de logs gerais
  let chat_logs_apro = client.channels.cache.get("1039626431937921096"); // Coloque o ID do chat de logs aprovados
  let chat_logs_repro = client.channels.cache.get("1039626431937921097"); // Coloque o ID do chat de logs reprovados
  let cor_embeds = "FF0000" // Coloque o código da cor para as embeds
  if (!cor_embeds) cor_embeds = "Random";

  if (message.author.bot) {

    return; //username.replace(" ", "-").replace("!", "-").replace(".", "-").replace(",", "-").replace(";", "-").replace(":", "-").replace("@", "-").replace("#", "-").replace("$", "-").replace("%", "-").replace("¨", "-").replace("&", "-").replace("*", "-").replace("(", "-").replace(")", "-").replace("=", "-").replace("+", "-").replace("[", "-").replace("]", "-").replace("{", "-").replace("}", "-").replace("<", "-").replace(">", "-").replace("/", "-").replace("\\", "-").replace("?", "-").replace("^", "-").replace("~", "-").replace("`", "-").replace("´", "-").replace("|", "-").replace("!", "-").replace(".", "-").replace(",", "-").replace(";", "-").replace(":", "-").replace("@", "-").replace("#", "-").replace("$", "-").replace("%", "-").replace("¨", "-").replace("&", "-").replace("*", "-").replace("(", "-").replace(")", "-").replace("=", "-").replace("+", "-").replace("[", "-").replace("]", "-").replace("{", "-").replace("}", "-").replace("<", "-").replace(">", "-").replace("/", "-").replace("\\", "-").replace("?", "-").replace("^", "-").replace("~", "-").replace("`", "-").replace("´", "-").replace("|", "-").toLocaleLowerCase()

  } else if (message.channel.name === `📝┃wl-${message.author.id}`) {

    setTimeout(() => {
      message.delete().catch(e => { })
    }, 1000)

    let first_msg = await db.get(`first_msg_${message.author.id}`)
    if (first_msg === false || first_msg === null) {

      await db.set(`first_msg_${message.author.id}`, true)

    setTimeout(async () => {
      message.channel.delete().catch(e => { });
      await db.delete(`first_msg_${message.author.id}`)
    }, 300000) // 5 minutos

      let msg_id = await db.get(`wl_${message.author.id}`);

      let chat = message.channel;

      let embed_1 = new Discord.EmbedBuilder()
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [02]__`)
        .setDescription(`> **Qual é o seu ID no jogo?**\n\n\\⏰ Você possui 5 minutos para completar tudo.`);

      let embed_2 = new Discord.EmbedBuilder()
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [03]__`)
        .setDescription(`> **Quantos anos você possui?**\n\n\\⏰ Você possui 5 minutos para completar tudo.`);

      let embed_3 = new Discord.EmbedBuilder() // Coloque a resposta no número 3
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [04]__`)
        .setDescription(`> **O que é PowerGaming?**

1️⃣ Atropelar as pessoas na cidade.
2️⃣ Fazer no jogo algo que pode ser feito na vida real.
3️⃣ Fazer no jogo algo que não pode ser feito na vida real
4️⃣ Matar as pessoas na cidade. 

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);

      let embed_4 = new Discord.EmbedBuilder()  // Coloque a resposta no número 1
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [05]__`)
        .setDescription(`> **O que é MetaGaming?**

1️⃣ Utilizar informações que não foram obtidas através do RP.
2️⃣ É andar sobre montanhas com carros que não fariam isso. 
3️⃣ É chamar personagem pelo nome. 
4️⃣ Nenhuma das afirmações anteriores.

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);

      let embed_5 = new Discord.EmbedBuilder()  // Coloque a resposta no número 4
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [06]__`)
        .setDescription(`> **O que é Anti-RP?**

1️⃣ É conversar com outro jogador. 
2️⃣ É cometer infração de trânsito. 
3️⃣ É desenvolver um papel da vida real. 
4️⃣ É não desenvolver um papel da vida real.

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);

      let embed_6 = new Discord.EmbedBuilder()  // Coloque a resposta no número 2
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [07]__`)
        .setDescription(`> **O que é Safezone?**

1️⃣ São lugares onde vende coletes.
2️⃣ São lugares em que não se pode roubar ou matar.
3️⃣ São lugares para regenerar vida.
4️⃣ São lugares onde se pode roubar e matar.

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);

      let embed_7 = new Discord.EmbedBuilder()  // Coloque a resposta no número 4
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [08]__`)
        .setDescription(`> **O que é VDM?**

1️⃣ Cair de moto em movimento.
2️⃣ É matar em lugares proibidos.
3️⃣ É dar carona de carro para outro jogador
4️⃣ É matar outro jogador atropelado.

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);

      let embed_8 = new Discord.EmbedBuilder()  // Coloque a resposta no número 2
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [09]__`)
        .setDescription(`> **O que é RDM?**

1️⃣ É matar um jogador porque ele tentou te roubar.
2️⃣ É matar um jogador sem motivo.
3️⃣ É matar um jogador porque ele roubou seu carro.
4️⃣ É matar um jogador porque ele tentou te matar.

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda digitando o número correspondente à resposta correta.`);

      let embed_9 = new Discord.EmbedBuilder()  // Coloque a resposta no número 3
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [10]__`)
        .setDescription(`> **Em qual você se destaca mais?**

➡ Farm.
➡ Tiro/Ação.
➡ Piloto.

\n\\⏰ Você possui 5 minutos para completar tudo.\n\\💬 Responda escrevendo uma das opções.`);

      let embed_10 = new Discord.EmbedBuilder()  // Coloque a resposta no número 1
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [11]__`)
        .setDescription(`> **Você já foi de alguma família?**

➡ Responda com Sim ou Não.
➡ Caso sim, qual família?

\n\\⏰ Você possui 5 minutos para completar tudo.`);

      let embed_11 = new Discord.EmbedBuilder()  // Coloque a resposta no número 1
        .setColor(cor_embeds)
        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
        .setTitle(`__Pergunta [12]__`)
        .setDescription(`> **Qual o número do seu celular no jogo?**

\n\\⏰ Você possui 5 minutos para completar tudo.`);

      chat.messages.edit(msg_id, { embeds: [embed_1] }).then(msg => {

        let filter = m => m.author.id == message.author.id;
        let coletor_1 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

        coletor_1.on('collect', async m1 => {

          let op1 = message.content;


          chat.messages.edit(msg_id, { embeds: [embed_2] }).then(msg => {

            let filter = m => m.author.id == message.author.id;
            let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

            coletor_2.on('collect', async m2 => {

              let op2 = m1.content;


              chat.messages.edit(msg_id, { embeds: [embed_3] }).then(msg => {

                let filter = m => m.author.id == message.author.id;
                let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                coletor_2.on('collect', async m3 => {

                  let op3 = m2.content;


                  chat.messages.edit(msg_id, { embeds: [embed_4] }).then(msg => {

                    let filter = m => m.author.id == message.author.id;
                    let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                    coletor_2.on('collect', async m4 => {

                      let op4 = m3.content;


                      chat.messages.edit(msg_id, { embeds: [embed_5] }).then(msg => {

                        let filter = m => m.author.id == message.author.id;
                        let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                        coletor_2.on('collect', async m5 => {

                          let op5 = m4.content;

                          chat.messages.edit(msg_id, { embeds: [embed_6] }).then(msg => {

                            let filter = m => m.author.id == message.author.id;
                            let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                            coletor_2.on('collect', async m6 => {

                              let op6 = m5.content;


                              chat.messages.edit(msg_id, { embeds: [embed_7] }).then(msg => {

                                let filter = m => m.author.id == message.author.id;
                                let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                                coletor_2.on('collect', async m7 => {

                                  let op7 = m6.content;


                                  chat.messages.edit(msg_id, { embeds: [embed_8] }).then(msg => {

                                    let filter = m => m.author.id == message.author.id;
                                    let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                                    coletor_2.on('collect', async m8 => {

                                      let op8 = m7.content;


                                      chat.messages.edit(msg_id, { embeds: [embed_9] }).then(msg => {

                                        let filter = m => m.author.id == message.author.id;
                                        let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                                        coletor_2.on('collect', async m9 => {

                                          let op9 = m8.content;


                                          chat.messages.edit(msg_id, { embeds: [embed_10] }).then(msg => {

                                            let filter = m => m.author.id == message.author.id;
                                            let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                                            coletor_2.on('collect', async m10 => {

                                              let op10 = m9.content;


                                              chat.messages.edit(msg_id, { embeds: [embed_11] }).then(msg => {

                                                let filter = m => m.author.id == message.author.id;
                                                let coletor_2 = chat.createMessageCollector({ filter: filter, max: 1, time: 60000 });

                                                coletor_2.on('collect', async m11 => {

                                                  let op11 = m10.content;

                                                      chat.messages.edit(msg_id, {
                                                        embeds: [
                                                          new Discord.EmbedBuilder()
                                                            .setColor(cor_embeds)
                                                            .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
                                                            .setTitle(`WhiteList Encerrada.`)
                                                            .setDescription(`> ${message.author} **Sua WhiteList chegou ao final.**`)
                                                        ]
                                                      })

                                                      let op12 = m11.content;

                                                      let quatro = op4
                                                      let cinco = op5
                                                      let seis = op6
                                                      let sete = op7
                                                      let oito = op8
                                                      let nove = op9

                                                      if (quatro !== "3") quatro = 0;
                                                      if (cinco !== "1") cinco = 0;
                                                      if (seis !== "4") seis = 0;
                                                      if (sete !== "2") sete = 0;
                                                      if (oito !== "4") oito = 0;
                                                      if (nove !== "2") nove = 0;

                                                      if (quatro === "3") quatro = 1; // Resposta certa 3
                                                      if (cinco === "1") cinco = 1; // Resposta certa 1
                                                      if (seis === "4") seis = 1; // Resposta certa 4
                                                      if (sete === "2") sete = 1; // Resposta certa 2
                                                      if (oito === "4") oito = 1; // Resposta certa 4
                                                      if (nove === "2") nove = 1; // Resposta certa 2

                                                      let questoes_acertos_e_erros = quatro + cinco + seis + sete + oito + nove;

                                                      await db.set(`user_wl_${message.author.id}`, true)

                                                      message.channel.send(`${message.author} Você terminou de responder sua WhiteList, veja os resultados em ${chat_logs}!\n${message.author} Este canal de texto será encerrado em 10 segundos.`);
                                                      setTimeout(() => {
                                                        message.channel.delete()
                                                      }, 10000)

                                                      let aprovado_ou_reprovado = ""
                                                      let cor_embed_log = ""

                                                      if (questoes_acertos_e_erros >= 4) {
                                                        aprovado_ou_reprovado = "Aprovado"
                                                        cor_embed_log = "Green"
                                                      }

                                                      if (questoes_acertos_e_erros < 4) {
                                                        aprovado_ou_reprovado = "Reprovado"
                                                        cor_embed_log = "Red"
                                                      }

                                                      let rep04 = ""
                                                      if (op4 === "1") rep04 = "Atropelar as pessoas na cidade."
                                                      if (op4 === "2") rep04 = "Fazer no jogo algo que pode ser feito na vida real."
                                                      if (op4 === "3") rep04 = "Fazer no jogo algo que não pode ser feito na vida real."
                                                      if (op4 === "4") rep04 = "Matar as pessoas na cidade."
                                                      if (op4 !== "1" && op4 !== "2" && op4 !== "3" && op4 !== "4") rep04 = "Alternativa inexistente."
                                                      let correct04 = "3"
                                                      let em04 = ""
                                                      if (correct04 === op4) {
                                                        em04 = "✅"
                                                      } else if (correct04 !== op4) {
                                                        em04 = "❌"
                                                      }

                                                      let rep05 = ""
                                                      if (op5 === "1") rep05 = "Utilizar informações que não foram obtidas através do RP."
                                                      if (op5 === "2") rep05 = "É andar sobre montanhas com carros que não fariam isso."
                                                      if (op5 === "3") rep05 = "É chamar personagem pelo nome."
                                                      if (op5 === "4") rep05 = "Nenhuma das afirmações anteriores."
                                                      if (op5 !== "1" && op5 !== "2" && op5 !== "3" && op5 !== "4") rep05 = "Alternativa inexistente."
                                                      let correct05 = "1"
                                                      let em05 = ""
                                                      if (correct05 === op5) {
                                                        em05 = "✅"
                                                      } else if (correct05 !== op5) {
                                                        em05 = "❌"
                                                      }

                                                      let rep06 = ""
                                                      if (op6 === "1") rep06 = "É conversar com outro jogador."
                                                      if (op6 === "2") rep06 = "É cometer infração de trânsito."
                                                      if (op6 === "3") rep06 = "É desenvolver um papel da vida real."
                                                      if (op6 === "4") rep06 = "É não desenvolver um papel da vida real."
                                                      if (op6 !== "1" && op6 !== "2" && op6 !== "3" && op6 !== "4") rep06 = "Alternativa inexistente."
                                                      let correct06 = "4"
                                                      let em06 = ""
                                                      if (correct06 === op6) {
                                                        em06 = "✅"
                                                      } else if (correct06 !== op6) {
                                                        em06 = "❌"
                                                      }

                                                      let rep07 = ""
                                                      if (op7 === "1") rep07 = "São lugares onde vende coletes."
                                                      if (op7 === "2") rep07 = "São lugares em que não se pode roubar ou matar."
                                                      if (op7 === "3") rep07 = "São lugares para regenerar vida."
                                                      if (op7 === "4") rep07 = "São lugares onde se pode roubar e matar."
                                                      if (op7 !== "1" && op7 !== "2" && op7 !== "3" && op7 !== "4") rep07 = "Alternativa inexistente."
                                                      let correct07 = "2"
                                                      let em07 = ""
                                                      if (correct07 === op7) {
                                                        em07 = "✅"
                                                      } else if (correct07 !== op7) {
                                                        em07 = "❌"
                                                      }

                                                      let rep08 = ""
                                                      if (op8 === "1") rep08 = "Cair de moto em movimento."
                                                      if (op8 === "2") rep08 = "É matar em lugares proibidos."
                                                      if (op8 === "3") rep08 = "É dar carona de carro para outro jogador"
                                                      if (op8 === "4") rep08 = "É matar outro jogador atropelado."
                                                      if (op8 !== "1" && op8 !== "2" && op8 !== "3" && op8 !== "4") rep08 = "Alternativa inexistente."
                                                      let correct08 = "4"
                                                      let em08 = ""
                                                      if (correct08 === op8) {
                                                        em08 = "✅"
                                                      } else if (correct08 !== op8) {
                                                        em08 = "❌"
                                                      }

                                                      let rep09 = ""
                                                      if (op9 === "1") rep09 = "É matar um jogador porque ele tentou te roubar."
                                                      if (op9 === "2") rep09 = "É matar um jogador sem motivo."
                                                      if (op9 === "3") rep09 = "É matar um jogador porque ele roubou seu carro."
                                                      if (op9 === "4") rep09 = "É matar um jogador porque ele tentou te matar."
                                                      if (op9 !== "1" && op9 !== "2" && op9 !== "3" && op9 !== "4") rep09 = "Alternativa inexistente."
                                                      let correct09 = "2"
                                                      let em09 = ""
                                                      if (correct09 === op9) {
                                                        em09 = "✅"
                                                      } else if (correct09 !== op9) {
                                                        em09 = "❌"
                                                      }

                                                      let embed_logs = new Discord.EmbedBuilder()
                                                        .setColor(cor_embed_log)
                                                        .setAuthor({ name: message.guild.name, iconURL: message.guild.iconURL({ dynamic: true }) })
                                                        .setThumbnail(message.author.displayAvatarURL({ dynamic: true })) // Fotinha do lado na direita em cima na embed de log da wl
                                                        .addFields(
                                                          {
                                                            name: `📌 Usuário:`,
                                                            value: `${message.author}.`,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Status:`,
                                                            value: `\`${aprovado_ou_reprovado}\`.`,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Nome:`,
                                                            value: `\`${op1}\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 ID:`,
                                                            value: `\`${op2}\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Idade:`,
                                                            value: `\`${op3}\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Atividade que o usuário mais se destaca:`,
                                                            value: `\`${op10}\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Já fez parte de alguma família?`,
                                                            value: `\`${op11}\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Número de celular no jogo:`,
                                                            value: `\`${op12}\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Perguntas acertadas:`,
                                                            value: `\`${questoes_acertos_e_erros}/6\``,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `📌 Perguntas:`,
                                                            value: `**O que é PowerGaming?**\n\`${rep04}\` ${em04}\n**O que é MetaGaming?**\n\`${rep05}\` ${em05}\n** O que é Anti-RP?**\n\`${rep06}\` ${em06}\n**O que é Safezone?**\n\`${rep07}\` ${em07}\n**O que é VDM?**\n\`${rep08}\` ${em08}\n**O que é RDM?**\n\`${rep09}\` ${em09}`,
                                                            inline: false
                                                          }
                                                        );

                                                      chat_logs.send({ embeds: [embed_logs] }).then(async () => {

                                                        await db.delete(`first_msg_${message.author.id}`)

                                                        if (questoes_acertos_e_erros >= 4) { // Log aprovado

                                                          let cargo = "1039626431518482474" // Coloque o ID do cargo
                                                          message.member.roles.add(cargo)

                                                          message.member.setNickname(`${op1} | ${op2}`)

                                                          let embed = new Discord.EmbedBuilder()
                                                          .setColor(cor_embed_log)
                                                          .setFooter({ text: `Staff` })
                                                          .setTimestamp(new Date())
                                                          .setAuthor({ name: `Recrutamento - ${message.guild.name}`, iconURL: message.guild.iconURL({ dynamic: true }) })
                                                          .setTitle(`Resultado do Recrutamento`)
                                                          .addFields(
                                                          {
                                                            name: `USUÁRIO:`,
                                                            value: `${message.author}.`,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `SITUAÇÃO:`,
                                                            value: `Aprovado ✅`,
                                                            inline: false
                                                          }
                                                        );

                                                        chat_logs_apro.send({ embeds: [embed] })

                                                        } else { // Log reprovado

                                                          let embed = new Discord.EmbedBuilder()
                                                          .setColor(cor_embed_log)
                                                          .setFooter({ text: `Staff` })
                                                          .setTimestamp(new Date())
                                                          .setAuthor({ name: `Recrutamento - ${message.guild.name}`, iconURL: message.guild.iconURL({ dynamic: true }) })
                                                          .setTitle(`Resultado do Recrutamento`)
                                                          .addFields(
                                                          {
                                                            name: `USUÁRIO:`,
                                                            value: `${message.author}.`,
                                                            inline: false
                                                          },
                                                          {
                                                            name: `SITUAÇÃO:`,
                                                            value: `Reprovado ❌`,
                                                            inline: false
                                                          }
                                                        );

                                                        chat_logs_repro.send({ embeds: [embed] })
                                                        

                                                        }

                                                      })

                                                });

                                              })

                                            });

                                          })

                                        });

                                      })

                                    });

                                  })

                                });

                              })

                            });

                          })

                        });

                      })

                    });

                  })

                });

              })

            });

          })

        });

      })
    }

  }
})

client.on("guildMemberAdd", (member) => {
  let channel = member.guild.channels.cache.get("1039626431543652460");
  let embed = new Discord.EmbedBuilder()
  .setColor("Green")
  .setAuthor({ name: `${member.user.tag} | Bem-Vindo(a)!`, iconURL: member.user.displayAvatarURL({ dynamic: true }) })
  .setThumbnail(member.user.displayAvatarURL({ dynamic: true }))
  .setDescription(`Olá, seja bem-vindo(a) ao servidor ${member.guild.name}!`)
  .addFields(
      {
          name: `💡 Curiosidade...`,
          value: `Você é o ${member.guild.memberCount}º membro aqui no servidor!`,
          inline: true
      },
      {
          name: `🛡 Tag do Usuário`,
          value: `\`${member.user.tag}\`\n(${member.id})`,
          inline: true
      },
      {
          name: `👮‍♂️ Evite punições!`,
          value: `Leia o chat <#1039626432206348303> para evitar ser punido no servidor!`,
          inline: true
      },
      {
          name: `Olá, seja bem-vindo(a) ao servidor ${member.guild.name}!!`,
          value: `⚪ | [Instagram](https://instagram.com/).\n🔵 | [Twitter](https://twitter.com/).\n🟣 | [Twitch](https://www.twitch.tv/).\n🔴 | [YouTube](https://www.youtube.com/channel/).\n⚫ | [TikTok](https://www.tiktok.com/@).`,
          inline: false
      }
  )
  .setFooter({ text: `${member.guild.name} • © Todos os direitos reservados.`, iconURL: member.guild.iconURL({ dynamic: true }) });

  channel.send({ content: `${member}`, embeds: [embed] });
});

client.on("guildMemberRemove", (member) => {
  let channel = member.guild.channels.cache.get("1039626431543652461");

  let embed = new Discord.EmbedBuilder()
  .setThumbnail(member.user.displayAvatarURL({ dynamic: true }))
  .setColor("Red")
  .setAuthor({ name: member.guild.name, iconURL: member.guild.iconURL({ dynamic: true }) })
  .setDescription(`O usuário ${member} saiu do servidor!\n\n> Id: \`${member.id}\`.\n> Tag: \`${member.user.tag}\`.`);

  channel.send({ embeds: [embed] });
});

client.on("interactionCreate", async (interaction) => {
  if (interaction.isButton()) {
    if (interaction.customId === "tickets_basico") {
      let nome_canal = `🔖-${interaction.user.id}`;
      let canal = interaction.guild.channels.cache.find(c => c.name === nome_canal);

      if (canal) {
        interaction.reply({ content: `Olá **${interaction.user.username}**, você já possui um ticket em ${canal}.`, ephemeral: true})
      } else {

        let categoria = interaction.channel.parent;
        if (!categoria) categoria = null;

        interaction.guild.channels.create({

          name: nome_canal,
          parent: categoria,
          type: Discord.ChannelType.GuildText,
          permissionOverwrites: [
            {
              id: interaction.guild.id,
              deny: [ Discord.PermissionFlagsBits.ViewChannel ]
            },
            {
              id: interaction.user.id,
              allow: [
                Discord.PermissionFlagsBits.ViewChannel,
                Discord.PermissionFlagsBits.AddReactions,
                Discord.PermissionFlagsBits.SendMessages,
                Discord.PermissionFlagsBits.AttachFiles,
                Discord.PermissionFlagsBits.EmbedLinks
              ]
            },
          ]

        }).then( (chat) => {

          interaction.reply({ content: `Olá **${interaction.user.username}**, seu ticket foi aberto em ${chat}.`, ephemeral: true })

          let embed = new Discord.EmbedBuilder()
          .setColor("Random")
          .setDescription(`Olá ${interaction.user}, você abriu o seu ticket.\nAguarde um momento para ser atendido.`);

          let botao_close = new Discord.ActionRowBuilder().addComponents(
            new Discord.ButtonBuilder()
            .setCustomId("close_ticket")
            .setEmoji("🔒")
            .setStyle(Discord.ButtonStyle.Danger)
          );

          chat.send({ embeds: [embed], components: [botao_close] }).then(m => {
            m.pin()
          })

        })
      }
    } else if (interaction.customId === "close_ticket") {
      if (!interaction.member.permissions.has(Discord.PermissionFlagsBits.Administrator)) return interaction.reply({ content: `Olá ${interaction.user}, você não possui permissão de fechar o seu ticket.` })
      interaction.reply(`Olá ${interaction.user}, este ticket será excluído em 5 segundos.`)
      try {
        setTimeout( () => {
          interaction.channel.delete().catch( e => { return; } )
        }, 5000)
      } catch (e) {
        return;
      }
      
    }
  }
})
