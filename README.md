# Final-project-

diffculties- 
Coming up with ideas to make it more 'complex'
Learning how APIs work and how to work with them
Discord reaction commands
Organization of data and getting the data to be displayed how we want it to be displayed
getting the tokens to work 

learnings-
bot discord and  api



import os
import random
import discord
from datetime import datetime
from discord.ext import commands
import requests



bot = commands.Bot(command_prefix='-')

@bot.event
async def on_ready():
    print('We have logged in as')


@bot.command(name='commands', help='Displays a list of commands')
async def commands(ctx):
    embed = discord.Embed(
        title="Commands",
        description="List of Commands")

    embed.add_field(name = "-hi", value="Replies with Hi",inline =False)
    embed.add_field(name = "-roll", value="Rolls a dice, must add 2 arguments\narg1 = number of rolls\narg2 = number of faces", inline=False)
    embed.add_field(name = "-weather current", value="enter the city that you want the wether on",inline =False)
    embed.add_field(name = "-weather week", value="enter the city that you want the wether on and get info on it for 1week",inline =False)
    embed.add_field(name= "-rockpaper", value= "must add an argument thats rock,paper,scissor",inline=False)
    embed.add_field(name= "-P", value= "must add give an reactions thats either thumbs up or down\narg1 = thumbs up for encourages\narg2 = thumbs down for insults",inline=False)

    embed.add_field(name="-islive",value="put in the user name of any streamer on twitch for inf")
    await ctx.send(embed=embed)
    

@bot.command(name='p')
async def hi(ctx):
    msg = await ctx.send('What a day!')
    await msg.add_reaction("\N{THUMBS UP SIGN}")
    await msg.add_reaction("\N{THUMBS DOWN SIGN}")

    def check(reaction, user):
        if(user != bot.user):
            return str(reaction.emoji) == "\N{THUMBS UP SIGN}" or str(reaction.emoji) == "\N{THUMBS DOWN SIGN}"
        else:
            return False

    try:
        reaction, user = await bot.wait_for('reaction_add', timeout=10, check=check)

        insults = [" is a baddie!", " is a loser!", " is a poopy head!"]
        encouragements = [" is so hot!", " is the best!", " is so cool!"]

        randNum = random.randint(0,2)

        if(str(reaction.emoji) == "\N{THUMBS UP SIGN}"):
            await ctx.send(format(ctx.author.display_name + encouragements[randNum]))

        if(str(reaction.emoji) == "\N{THUMBS DOWN SIGN}"):
            await ctx.send(format(ctx.author.display_name + insults[randNum]))

    except asyncio.TimeoutError:
        #await ctx.send("NO TIME")
        pass



@bot.command(name='islive')
async def islive(ctx, username):
    CLIENT_ID = 'allo8fj1j52dinmn47tbraex41gieb'
    OAUTH_TOKEN = '1x66cch9j2hg9275osyormwe2q47ob'

    endpoint = 'https://api.twitch.tv/helix/streams'
    my_headers = {
        'Client-ID': CLIENT_ID,
        'Authorization': f'Bearer {OAUTH_TOKEN}'
    }
    my_params = {'user_login':username}
    response = requests.get(endpoint,headers=my_headers, params=my_params)

    data = response.json()['data']

    if len(data) == 0:
        await ctx.send("This user does not exist or is not live!")
    else:
        resJSON = data[0]
        print(resJSON)
        embed = discord.Embed(
            title=resJSON['title'],
            url='https://www.twitch.tv/' + resJSON['user_login'],
            description=resJSON['user_name'] + ' is currently live playing ' + resJSON['game_name'] + '!'
        )
        embed.add_field(name='Viewer Count', value=resJSON['viewer_count'])
        await ctx.send(embed=embed)


        await ctx.send('https://www.twitch.tv/' + resJSON['user_login'])




@bot.command(name='hi')
async def hi(ctx):
    await ctx.send('hi')



@bot.command(name='roll')
async def roll(ctx, number_of_dice: int, number_of_sides: int):
    dice = [
        str(random.choice(range(1, number_of_sides + 1)))
        for _ in range(number_of_dice)
    ]
    await ctx.send(', '.join(dice))



@bot.command(name='rockpaper')
async def paper(ctx):
    rockpaper = ['rock', 'paper', 'scissor']
    await ctx.send(random.choice(rockpaper))




@bot.command(name='weather')
async def weather(ctx, reqType, *, city: str):
    api_key = "fc7972be5763e24282efd30a4e8f296d"
    base_url = "http://api.openweathermap.org/data/2.5/weather?"

    response = requests.get(base_url + "appid=" + api_key + "&q=" + city)
    resJSON = response.json()

    if reqType == "current":
        embed = discord.Embed(
            title=resJSON['name'] + '\'s Weather'
        )

        embed.add_field(name="Current Weather", value=resJSON['weather'][0]['main'])
        embed.add_field(name="Description", value=resJSON['weather'][0]['description'])

        mainJSON = resJSON['main']
        temp = mainJSON['temp']
        celsius = temp-273.15
        fahrenheit = celsius * (9/5) + 32

        embed.add_field(name="Temperature(F)", value=f"{str(round(fahrenheit))}°F", inline=False)
        embed.add_field(name="Temperature(C)", value=f"{str(round(celsius))}°C", inline=True)
        embed.add_field(name="Humidity(%)", value=f"**{mainJSON['humidity']}%**", inline=False)

        embed.set_thumbnail(url='http://openweathermap.org/img/w/' + resJSON['weather'][0]['icon'] + '.png')
        await ctx.send(embed=embed)
    elif reqType == "week":
        embed = discord.Embed(
            title=resJSON['name'] + '\'s 7-day Weather'
        )
        embed.set_thumbnail(url="https://i.ibb.co/CMrsxdX/weather.png")
        lat = resJSON['coord']['lat']
        lon = resJSON['coord']['lon']

        week_url = "https://api.openweathermap.org/data/2.5/onecall?"
        response = requests.get(week_url + "appid=" + api_key + "&lat=" + str(lat) + "&lon=" + str(lon))
        dailyArr = response.json()['daily']

        for dailyJSON in dailyArr:
            utcTime = dailyJSON['dt']
            time = datetime.utcfromtimestamp(utcTime).strftime('%Y-%m-%d')
            temp = (dailyJSON['temp']['min'] + dailyJSON['temp']['max'])/2
            celsius = temp-273.15
            fahrenheit = celsius * (9/5) + 32
            if dailyJSON['weather'][0]['main'] == 'Clear':
                weather = '\☀️ Clear'
            elif dailyJSON['weather'][0]['main'] == "Clouds":
                weather = '\☁️ Clouds'
            elif dailyJSON['weather'][0]['main'] == "Rain":
                weather = '\🌧️ Rain'
            elif dailyJSON['weather'][0]['main'] == "Thunderstorm":
                weather = '\🌩️ Thunderstorm'
            elif dailyJSON['weather'][0]['main'] == "Drizzle":
                weather = '\🌧️ Drizzle'
            embed.add_field(name=time, value = weather + f'\n{str(round(fahrenheit))}°F || {str(round(celsius))}°C', inline=False)
        await ctx.send(embed=embed)
    else:
        await ctx.send("Please specify 'current' or 'week' before the city.")



bot.run('ODM0OTY2NDU0NDczNzE5ODM4.YIIlGw.e0tVu_dh9MnncrH9ZE3BsVNrAkk')

