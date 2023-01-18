# inform-message-6

```

from enum import Enum
import os
import discord
from discord.ext import commands
import asyncio
import pathlib
import ffmpeg
import base64
import requests
from wit import Wit
import io
import speech_recognition as s_r
client = Wit("3JFZZNUEAANTF76DOPHREBCMGFYAO32I")
token='token'
bot = commands.Bot(command_prefix = ["."], intents = discord.Intents.all(), help_command = None) # тут можно задать префикс
connections = {}
api_token = 'my_api_token'
api_url = 'https://api.silero.ai/transcribe'


r = s_r.Recognizer()



@bot.event
async def on_ready():
    print("Bot - Online..") # эта функция воспроизводится один раз при запуске бота, после которой бот будет включен и будет реагировать на команды

@bot.command()
async def start(ctx): # эта функция вызывается каждый когда мы пишем в текстовый чат команду ".start", т.к. у нас функция называется "start", а префикс "."
    if ctx.message.author.voice: # проверка есть ли человек который вызвал эту команду в войс канале
        voice = ctx.author.voice
        channel = ctx.author.voice.channel
        voice1 = ctx.message.guild.voice_client
        if not voice1: # проверка нет ли бот в войс канале
            vc = await voice.channel.connect()  # присоединение в войс канал
            connections.update({ctx.guild.id: vc}) 
            print("Старт в канале '" + channel.name + "'")
            vc.start_recording( # начало записи
                discord.sinks.WaveSink(), 
                once_done,  
                ctx.channel 
            )
            print("Запись начата в канале '" + channel.name + "'")
        else:
            if channel.id != voice1.channel.id: # проверкка на то, что бот и человек не в разынх каналах
                name_from = voice1.channel.name
                if ctx.guild.id in connections: # этот иф был в примере и оно так работает
                    vc = connections[ctx.guild.id]
                    vc.stop_recording() # конец записи
                    del connections[ctx.guild.id] 
                    print("Стоп в канале '" + name_from + "'")
                    print("Стоп записи в канале '" + name_from + "'")
                    await asyncio.sleep(2)
                    vc = await voice.channel.connect()  
                    print("Перемещение из канала '" + name_from + "' в канал '" + channel.name + "'") # перемещение из одного войс канала в другой с сохранением записи
                    connections.update({ctx.guild.id: vc})  
                    vc.start_recording( # начало записис
                        discord.sinks.WaveSink(),  
                        once_done,  
                        ctx.channel 
                    )
                    print("Запись начата в канале '" + channel.name + "'")
                else:
                    print("Ошибка")
                #print("Начало записи в канале '" + channel.name + "'")
            else:
                print("Бот уже в этом канале - '" + channel.name + "'")
    else:
        print("Автора команды нет в голосовом канале")

async def once_done(sink: discord.sinks, channel: discord.TextChannel, *args): # функция куда идет аудиофайл, а также его последующая обработка
    recorded_users = [
        f"<@{user_id}>"
        for user_id, audio in sink.audio_data.items()
    ]
    await sink.vc.disconnect() 
    files_out = [(audio.file, f"{user_id}.{sink.encoding}") for user_id, audio in sink.audio_data.items()]
    #files = [discord.File(audio.file, f"{user_id}.{sink.encoding}") for user_id, audio in sink.audio_data.items()]
    #print(files[0])
    #a = files[0]
    print(files_out[0], "\n\n\n\n\n\n\n\n\n\n\n")
    with s_r.AudioFile(files_out[0][0]) as to_text:
        data = r.record(to_text)
        textt = r.recognize_google(data, language = 'ru-RU', show_all = True)
        print(textt)
    files = []
    files.append(discord.File(files_out[0][0], files_out[0][1]))
    #text = client.speech(a[0], {'Content-Type': 'audio/wav'})
    print(files)
    await channel.send(textt, files=files)  

@bot.command()
async def stop(ctx): # эта функция вызывается каждый когда мы пишем в текстовый чат команду ".stop", т.к. у нас функция называется "stop", а префикс "."
    channel = ctx.author.voice.channel
    voice = ctx.message.guild.voice_client
    if voice: # проверка есть ли бот в войс канале
        if ctx.author.voice: # есть ли автор команды в войс канале
            if channel.id != voice.channel.id: # проверка на то, в разных ли каналах бот и человек который вызвал команду
                print("Бот находится в другом канале - '" + voice.channel.name + "'")
            else:
                if ctx.guild.id in connections: # этот иф был в примере и оно так работает
                    vc = connections[ctx.guild.id]
                    vc.stop_recording() # стоп записи
                    del connections[ctx.guild.id]  
                    print("Стоп в канале '" + channel.name + "'")
                    print("Стоп записи в канале '" + channel.name + "'")
                else:
                    print("Ошибка")
        else:
            print("Автор сообщения не находится в голосовом канале")
    else:
        print("Бот не играет")

@bot.command()
async def info(ctx): # тестовая функция на вывод всех данных который мы можем вообще получить о людях который есть в войс канале вместе с ботом
    i = 0
    if ctx.message.guild.voice_client != None:
        for i in range(len(ctx.message.guild.voice_client.channel.members)):
            print(str(ctx.message.guild.voice_client.channel.members[i]) + " - " + str(ctx.message.guild.voice_client.channel.members[i].voice + "\n================================"))
            i += 1
    else:
        print("Бот не играет")

bot.run(token) # старт бота

```
