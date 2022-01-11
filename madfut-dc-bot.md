import discord
from discord.ext import commands
import random
from discord import Permissions
import asyncio
import datetime


client = commands.Bot (command_prefix=commands.when_mentioned_or ("="), help_command=None)

filtered_words = ("fuck", "kys", "cunt", "nigger", "nigga", "ass", "twat", "penis", "sex", "pussy")

@client.event
async def on_ready():
    print ('Bot Is Ready to go!') 

@client.event
async def on_message(msg):
    for word in filtered_words:
        if word in msg.content:
            await msg.delete()

    await client.process_commands(msg)

@client.command()
@commands.has_role('.')
async def shutdown(ctx):
    await ctx.send('Logged out :thumbsup:')
    await ctx.bot.logout()

@client.command()
async def messagerewards(ctx):
    embed = discord.Embed(title="Message Rewards", description="350: 3 cards of choice\n550:  6 cards of choice\n1k+:  12 custom packs & 12 cards of choice!") #,color=#0000FF
    await ctx.send(embed=embed)
    return

@client.command()
async def inviterewards(ctx):
    embed = discord.Embed(title="Invite Rewards", description="5 invites = 20m & 3 cards of choice\n10 invites = 35m & 5 cards of choice\n15 invites = 75m & 7 cards of choice\n20 invites = 95m & 10 cards of choice") #,color=#0000FF
    await ctx.send(embed=embed)

@client.command()
async def levelrewards(ctx):
    embed = discord.Embed(title="Level Rewards", description=" level 5 = 35m & 5 cards of choice\nlevel 10 =55m & 6 cards of choice\nlevel  20 = 65m & 11 cards of choice") #,color=#0000FF
    await ctx.send(embed=embed)
    return

@client.command()
async def boostrewards(ctx):
    embed = discord.Embed(title="Boost Rewards", description=" every boost is 250m, 10 modded packs and 10 cards of choice & 5 custom packs") #,color=#0000FF
    await ctx.send(embed=embed)
    return



@client.command(aliases=['h'])
async def help(ctx):
    embed = discord.Embed(title="Prefix: = Commands: ", description="Reward Commands:\ninviterewards\nlevelrewards\nboostrewards\nmessagerewards") #,color=#0000FF
    await ctx.send(embed=embed)
    return

@client.command(aliases=['ah'])
@commands.has_permissions(kick_members = True)
async def adminhelp(ctx):
    embed = discord.Embed(title="Prefix: = Admin Commands: ", description="Admin Commands: ban, unban\nmute, unmute\nkick\npurge") #,color=#0000FF
    await ctx.send(embed=embed)
    return

@client.command(aliases=['w'])
@commands.has_permissions(kick_members = True)
async def warn(ctx, member : discord.Member,*,reason= "No Reason provided"):
  if member == ctx.author:
    await ctx.send('You cannot warn yourself!')
  else:
    em = discord.Embed(title="Warned", description=f"{member.mention} was warned for {reason}", colour=discord.Color.blue())
    em2 = discord.Embed(title="Warned", description=f"You have been warned on Madfut Society for: {reason}", colour=discord.Color.blue())
    await member.send(embed=em2)
    await member.send(embed=em)


@client.command(aliases=['p'])
@commands.has_permissions(manage_messages = True)
async def purge(ctx, amount=2):
  await ctx.channel.purge(limit = amount)

@client.command(aliases=['k'])
@commands.has_permissions(kick_members = True)
async def kick(ctx, member : discord.Member,*,reason= "No Reason provided"):
  await ctx.send(member.name + ' has been kicked from Madfut Society because:' +reason)
  await member.kick(reason=reason)

@client.command(aliases=['b'])
@commands.has_permissions(ban_members = True)
async def ban(ctx, member : discord.Member,*,reason= "No Reason provided"):
  await ctx.send (member.name + ' has been banned from Madfut Society because:' +reason)
  await member.ban(reason=reason)
  return

@client.command()
@commands.has_permissions(ban_members = True)
async def unban(ctx, *, member):
  banned_users = await ctx.guild.bans()
  member_name, member_discriminator = member.split('#')

  for ban_entry in banned_users:
    user = ban_entry.user

    if (user.name, user.discriminator) == (member_name, member_discriminator):
      await ctx.guild.unban(user)
      await ctx.send('Unbanned :thumbsup:')
      return

@client.command(description="Unmutes a specified user.")
@commands.has_permissions(manage_messages=True)
async def unmute(ctx, member: discord.Member):
   mutedRole = discord.utils.get(ctx.guild.roles, name="Muted")

   await member.remove_roles(mutedRole)
   await member.send(f" you have unmuted from: - {ctx.guild.name}")
   embed = discord.Embed(title="unmute", description=f" unmuted-{member.mention}",colour=discord.Colour.light_gray())
   await ctx.send(embed=embed)
   return

@client.command(description="Mutes the specified user.")
@commands.has_permissions(manage_messages=True)
async def mute(ctx, member: discord.Member, *, reason=None):
    guild = ctx.guild
    mutedRole = discord.utils.get(guild.roles, name="Muted")

    if not mutedRole:
        mutedRole = await guild.create_role(name="Muted")

        for channel in guild.channels:
            await channel.set_permissions(mutedRole, speak=False, send_messages=False, read_message_history=True, read_messages=False)
    embed = discord.Embed(title="muted", description=f"{member.mention} was muted ", colour=discord.Colour.light_gray())
    embed.add_field(name="reason:", value=reason, inline=False)
    await ctx.send(embed=embed)
    await member.add_roles(mutedRole, reason=reason)
    await member.send(f" you have been muted by: {guild.name} reason: {reason}")
    return

user = client.get_user(int(654778104925126666))

@client.command()
@commands.has_role(".")
async def msg_groupez(ctx, user: discord.User, message):
    await user.send(message)
    await ctx.send('Sent a dm to <@!654778104925126666>')


@client.command()
@commands.has_role("giveaways")
async def giveaway(ctx):
    # Giveaway command requires the user to have a "giveaways" role to function properly

    # Stores the questions that the bot will ask the user to answer in the channel that the command was made
    # Stores the answers for those questions in a different list
    giveaway_questions = ['Which channel will I host the giveaway in?', 'What is the prize?', 'How long should the giveaway run for (in seconds)?',]
    giveaway_answers = []

    # Checking to be sure the author is the one who answered and in which channel
    def check(m):
        return m.author == ctx.author and m.channel == ctx.channel

    # Askes the questions from the giveaway_questions list 1 by 1
    # Times out if the host doesn't answer within 30 seconds
    for question in giveaway_questions:
        await ctx.send(question)
        try:
            message = await client.wait_for('message', timeout= 30.0, check= check)
        except asyncio.TimeoutError:
            await ctx.send('You didn\'t answer in time.  Please try again and be sure to send your answer within 30 seconds of the question.')
            return
        else:
            giveaway_answers.append(message.content)

    # Grabbing the channel id from the giveaway_questions list and formatting is properly
    # Displays an exception message if the host fails to mention the channel correctly
    try:
        c_id = int(giveaway_answers[0][2:-1])
    except:
        await ctx.send(f'You failed to mention the channel correctly.  Please do it like this: {ctx.channel.mention}')
        return

    # Storing the variables needed to run the rest of the commands
    channel = client.get_channel(c_id)
    prize = str(giveaway_answers[1])
    time = int(giveaway_answers[2])

    # Sends a message to let the host know that the giveaway was started properly
    await ctx.send(f'The giveaway for {prize} will begin shortly.\nPlease direct your attention to {channel.mention}, this giveaway will end in {time} seconds.')

    # Giveaway embed message
    give = discord.Embed(color = 0x2ecc71)
    give.set_author(name = f'GIVEAWAY TIME!', icon_url = 'https://i.imgur.com/VaX0pfM.png')
    give.add_field(name= f'{ctx.author.name} is giving away: {prize}!', value = f'React with 🎉 to enter!\n Ends in {round(time/60, 2)} minutes!', inline = False)
    end = datetime.datetime.utcnow() + datetime.timedelta(seconds = time)
    give.set_footer(text = f'Giveaway ends at {end} UTC!')
    my_message = await channel.send(embed = give)

    # Reacts to the message
    await my_message.add_reaction("🎉")
    await asyncio.sleep(time)

    new_message = await channel.fetch_message(my_message.id)

    # Picks a winner
    users = await new_message.reactions[0].users().flatten()
    users.pop(users.index(client.user))
    winner = random.choice(users)


    # Reveals the winne

    await ctx.send(f'🎉**Congtrats!🎉\n\n{winner.mention} has won the giveaway for {prize}**\n**Entrants: {len(users)}**\n**Thank you for entering!**')

@client.command()
@commands.has_role("giveaways")
async def reroll(ctx, channel: discord.TextChannel, id_ : int):
    # Reroll command requires the user to have a "Giveaway Host" role to function properly
    try:
        new_message = await channel.fetch_message(id_)
    except:
        await ctx.send("Incorrect id.")
        return

    # Picks a new winner
    users = await new_message.reactions[0].users().flatten()
    users.pop(users.index(client.user))
    winner = random.choice(users)

    # Announces the new winner to the server
    reroll_announcement = discord.Embed(color = 0xff2424)
    reroll_announcement.set_author(name = f'The giveaway was re-rolled by the host!', icon_url = 'https://i.imgur.com/DDric14.png')
    reroll_announcement.add_field(name = f'🥳 New Winner:', value = f'{winner.mention}', inline = False)
    await channel.send(embed = reroll_announcement)


async def on_message(self, message):
        with open("blacklist.txt") as bf:
            blacklist = [word.strip().lower() for word in bf.readlines()]
        bf.close()

        channel = message.channel
        for word in blacklist:
            if word in message.content:
                bot_message = await channel.send(f'{user.mention}, this word is not allowed here!')
                await message.delete()
                await asyncio.sleep(3)
                await bot_message.delete()
            else:
                print("Nothing")


client.run('OTIxMzIzMTA3MDMyMTI5NTM2.YbxPAg.ewsUwL9W4vFu4hAatoUUNXTR6KE')
