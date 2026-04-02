bot.py


import discord
from discord.ext import commands
import json
import os
import random
from datetime import datetime, timedelta

# Bot setup
intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix='-', intents=intents)

# Data files
BALANCE_FILE = 'balances.json'
BUNDLES_FILE = 'bundles.json'
FIXTURES_FILE = 'fixtures.json'
SHOP_FILE = 'shop.json'

# Initialize data files if they don't exist
def init_files():
    if not os.path.exists(BALANCE_FILE):
        with open(BALANCE_FILE, 'w') as f:
            json.dump({}, f)
    
    if not os.path.exists(BUNDLES_FILE):
        with open(BUNDLES_FILE, 'w') as f:
            json.dump({"active_bundles": []}, f)
    
    if not os.path.exists(FIXTURES_FILE):
        with open(FIXTURES_FILE, 'w') as f:
            json.dump({"fixtures": []}, f)
    
    if not os.path.exists(SHOP_FILE):
        with open(SHOP_FILE, 'w') as f:
            json.dump({"items": []}, f)

# Load/Save functions
def load_json(filename):
    with open(filename, 'r') as f:
        return json.load(f)

def save_json(data, filename):
    with open(filename, 'w') as f:
        json.dump(data, f, indent=2)

# Sample players and bundles
SAMPLE_PLAYERS = {
    "Rohit Sharma": {"rarity": "Legendary", "team": "Mumbai Indians", "value": 500},
    "Virat Kohli": {"rarity": "Legendary", "team": "Royal Challengers Bangalore", "value": 450},
    "Jasprit Bumrah": {"rarity": "Epic", "team": "Mumbai Indians", "value": 300},
    "KL Rahul": {"rarity": "Epic", "team": "Lucknow Super Giants", "value": 280}
}

@bot.event
async def on_ready():
    init_files()
    print(f'{bot.user} has landed!')
    print('Bot is ready!')

# Balance Commands
@bot.command(name='balance')
async def balance(ctx, member: discord.Member = None):
    user = member or ctx.author
    data = load_json(BALANCE_FILE)
    
    coins = data.get(str(user.id), {}).get('coins', 0)
    tickets = data.get(str(user.id), {}).get('tickets', 0)
    
    embed = discord.Embed(title=f"{user.display_name}'s Balance", color=0x00ff00)
    embed.add_field(name="🪙 Coins", value=f"{coins:,}", inline=True)
    embed.add_field(name="🎫 Tickets", value=f"{tickets:,}", inline=True)
    await ctx.send(embed=embed)

@bot.command(name='addbalance')
async def add_balance(ctx, member: discord.Member, coins: int = 0, tickets: int = 0):
    if not ctx.author.guild_permissions.administrator:
        return await ctx.send("❌ Only admins can use this command!")
    
    data = load_json(BALANCE_FILE)
    user_id = str(member.id)
    
    if user_id not in data:
        data[user_id] = {'coins': 0, 'tickets': 0}
    
    data[user_id]['coins'] += coins
    data[user_id]['tickets'] += tickets
    
    save_json(data, BALANCE_FILE)
    await ctx.send(f"✅ Added {abs(coins)} coins and {abs(tickets)} tickets to {member.mention}")

@bot.command(name='setbalance')
async def set_balance(ctx, member: discord.Member, coins: int, tickets: int):
    if not ctx.author.guild_permissions.administrator:
        return await ctx.send("❌ Only admins can use this command!")
    
    data = load_json(BALANCE_FILE)
    user_id = str(member.id)
    data[user_id] = {'coins': coins, 'tickets': tickets}
    save_json(data, BALANCE_FILE)
    
    await ctx.send(f"✅ Set {member.mention}'s balance to {coins} coins and {tickets} tickets")

# Bundle System
@bot.command(name='bundle')
async def open_bundle(ctx):
    data = load_json(BALANCE_FILE)
    bundles_data = load_json(BUNDLES_FILE)
    
    user_id = str(ctx.author.id)
    if user_id not in data or data[user_id]['tickets'] < 1:
        return await ctx.send("❌ You need at least 1 ticket to open a bundle! Use `-balance` to check.")
    
    # Deduct ticket
    data[user_id]['tickets'] -= 1
    save_json(data, BALANCE_FILE)
    
    # Generate random player
    player_name = random.choice(list(SAMPLE_PLAYERS.keys()))
    player_data = SAMPLE_PLAYERS[player_name]
    
    # Mark as owned (won't appear in other bundles)
    bundles_data["active_bundles"].append({
        "player": player_name,
        "owner": user_id,
        "timestamp": datetime.now().isoformat()
    })
    save_json(bundles_data, BUNDLES_FILE)
    
    embed = discord.Embed(title="🎁 Bundle Opened!", color=0xffd700)
    embed.add_field(name="🏆 You got:", value=f"**{player_name}**", inline=False)
    embed.add_field(name="⭐ Rarity", value=player_data['rarity'], inline=True)
    embed.add_field(name="🏏 Team", value=player_data['team'], inline=True)
    embed.add_field(name="💎 Value", value=f"{player_data['value']:,} coins", inline=True)
    
    await ctx.send(embed=embed)

@bot.command(name='activebundles')
async def active_bundles(ctx):
    bundles_data = load_json(BUNDLES_FILE)
    active = bundles_data.get("active_bundles", [])
    
    if not active:
        return await ctx.send("📦 No active bundles!")
    
    embed = discord.Embed(title="📦 Active Bundles", color=0x0099ff)
    for bundle in active[-5:]:  # Show last 5
        player = bundle['player']
        owner_id = bundle['owner']
        embed.add_field(
            name=player, 
            value=f"<@{owner_id}>", 
            inline=False
        )
    await ctx.send(embed=embed)

# Fixtures System
@bot.command(name='fixtures')
async def fixtures(ctx):
    data = load_json(FIXTURES_FILE)
    fixtures_list = data.get("fixtures", [])
    
    if not fixtures_list:
        return await ctx.send("📅 No fixtures scheduled!")
    
    embed = discord.Embed(title="📅 Upcoming Fixtures", color=0x00ff00)
    for fixture in fixtures_list:
        embed.add_field(
            name=fixture['match'],
            value=f"📅 {fixture['date']}\n👥 Players: {', '.join(fixture['players'][:3])}...",
            inline=False
        )
    await ctx.send(embed=embed)

@bot.command(name='addfixture')
async def add_fixture(ctx, date: str, *, match_info: str):
    if not ctx.author.guild_permissions.administrator:
        return await ctx.send("❌ Only admins can use this command!")
    
    data = load_json(FIXTURES_FILE)
    players = random.sample(list(SAMPLE_PLAYERS.keys()), min(5, len(SAMPLE_PLAYERS)))
    
    fixture = {
        "match": match_info,
        "date": date,
        "players": players,
        "timestamp": datetime.now().isoformat()
    }
    
    data["fixtures"].append(fixture)
    save_json(data, FIXTURES_FILE)
    
    await ctx.send(f"✅ Added fixture: **{match_info}** on {date}")

@bot.command(name='removefixture')
async def remove_fixture(ctx, index: int = None):
    if not ctx.author.guild_permissions.administrator:
        return await ctx.send("❌ Only admins can use this command!")
    
    data = load_json(FIXTURES_FILE)
    fixtures_list = data.get("fixtures", [])
    
    if index is None:
        return await ctx.send("❌ Please specify fixture index! Use `-fixtures` to see list.")
    
    if 0 <= index < len(fixtures_list):
        removed = fixtures_list.pop(index)
        save_json(data, FIXTURES_FILE)
        await ctx.send(f"✅ Removed fixture: **{removed['match']}**")
    else:
        await ctx.send("❌ Invalid fixture index!")

# Shop System
@bot.command(name='shop')
async def shop(ctx):
    data = load_json(SHOP_FILE)
    items = data.get("items", [])
    
    if not items:
        # Generate sample shop items
        shop_items = [
            {"name": "Rohit Sharma Pack", "price_coins": 200, "price_tickets": 0, "rarity": "Legendary"},
            {"name": "Bumrah Special", "price_coins": 150, "price_tickets": 0, "rarity": "Epic"},
            {"name": "Mystery Bundle", "price_coins": 0, "price_tickets": 3, "rarity": "Rare"}
        ]
        data["items"] = shop_items
        save_json(data, SHOP_FILE)
        items = shop_items
    
    embed = discord.Embed(title="🏪 Player Shop", color=0xff6600)
    for i, item in enumerate(items, 1):
        price = f"{item['price_coins']}🪙 / {item['price_tickets']}🎫"
        embed.add_field(
            name=f"{i}. {item['name']}",
            value=f"⭐ {item['rarity']} | **{price}**",
            inline=False
        )
    embed.set_footer(text="Use -buy <number> to purchase")
    await ctx.send(embed=embed)

@bot.command(name='buy')
async def buy(ctx, item_num: int):
    balance_data = load_json(BALANCE_FILE)
    shop_data = load_json(SHOP_FILE)
    
    user_id = str(ctx.author.id)
    items = shop_data.get("items", [])
    
    if item_num < 1 or item_num > len(items):
        return await ctx.send("❌ Invalid item number!")
    
    item = items[item_num - 1]
    user_balance = balance_data.get(user_id, {'coins': 0, 'tickets': 0})
    
    if user_balance['coins'] < item['price_coins'] or user_balance['tickets'] < item['price_tickets']:
        return await ctx.send("❌ Insufficient balance!")
    
    # Deduct balance
    user_balance['coins'] -= item['price_coins']
    user_balance['tickets'] -= item['price_tickets']
    balance_data[user_id] = user_balance
    save_json(balance_data, BALANCE_FILE)
    
    embed = discord.Embed(title="✅ Purchase Successful!", color=0x00ff00, description=f"**{item['name']}** purchased!")
    await ctx.send(embed=embed)

# Error handling
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.MissingRequiredArgument):
        await ctx.send("❌ Missing required argument!")
    elif isinstance(error, commands.BadArgument):
        await ctx.send("❌ Invalid argument!")
    else:
        await ctx.send(f"❌ An error occurred: {str(error)}")

# Run the bot
bot.run('MTQ4OTI3MDI2ODI3NzgyMTc2NA.Gx7ean.SiwaiQOmuDOAeCBpeO6vJeyMSw9EaO7QXl3WiM')  # Replace with your bot token
