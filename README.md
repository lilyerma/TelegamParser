# Telegram Channel Parsing

I was looking for a solution to get a list of the participants (subscribers) for a particular Telegram channel. Although you will find a lot of posts on the internet that say you can do this with a single call to the API, unfortunately, this is no longer the case. The Telethon API call returns only 200 participants.

I've found a workaround by this guy [here](https://okhlopkov.com/how-to-get-a-telegram-channel-subscribers-list-in-python/). So Daniil gets the full credit for this. I'm just making a note for myself not to lose this Python experience.

## Created Python Virtual Environment

It's preferable to install libraries in an environment for a specific project you work with. Before that, I had a lot of trouble installing it to the global environment.

Use code with caution.

```
python3 -m venv my-venv
source my-venv/bin/activate
pip install ipykernel
```


Now you have a folder named "my-venv" and it's going to be your working directory.

## Installed Telethon

Telethon is a library for writing Python programs that can interact with Telegram.

```
pip install telethon
```


This installs Telethon to my virtual environment "my-env".

## Launch Jupyter Notebook

Now launch a notebook to write code.

```
pip install jupyter
jupyter notebook
```


## Choose kernel "my-env"

In Jupyter notebook, you can choose the kernel to use. Choose the "my-env" kernel.

## Code Walk Through

### Imports

```
import telethon, itertools, asyncio, random, csv
from telethon import TelegramClient
import pandas as pd
```


We will send asynchronous requests with a random timeout so the API won't be blocked. That's why we need *asyncio, random*. *csv* is needed to send the results as a file. *Pandas* is needed to transform data with Python (e.g., I need special fields and filters). *itertools* will help iterate by options.

### Credentials

You get this by authorizing here https://my.telegram.org and getting to tools for developers.

```
api_id =
api_hash = ''
phone = ''
channel_href = ''
```

Now we initiate the Telethon client that will be making API calls.

```
client = TelegramClient('session_name', api_id, api_hash)
client = await client.start()
dialogs = await client.get_dialogs()
```

I feel you don't need the thing with all channels, but it was in the original solution, so I reserved it.

When you launch the code, it will ask for phone number, an SMS (Telegram) code and the password for your Telegram account.

#### The Main Solution

Although the recommended method to list all participants or even to provide participants by pages doesn't work, searching by participants can provide samples of users. So here comes an algorithm to iterate through alphabets.

Be aware that if your channel is popular enough, this code will take time to run. In my case, 10K+ participants took more than an hour (I wasn't keeping track of how long), but it yielded me 9,780 unique users, which is good enough.

```Python
def search_queries():
    alphabets = [
        "qwertyuiopasdfghjklzxcvbnm",
        "йцукенгшщзхъёфывапролджэячсмитьбю",
        "1234567890",
    ]
    
    for alphabet in alphabets:
        for c in alphabet:
            yield c 
    
    # if your channel is large (I haven't noticed this, so guess I had run both loops)
    # let's generate pairs:
    for alphabet in alphabets:
        for pair in itertools.permutations(alphabet, 2):
            yield "".join(pair)

participants = {}  # store users info here

for query in search_queries():
    async for res in client.iter_participants(
        channel, search=query
    ):
        participants[res.id] = res.to_dict() 
    
    # if you want to be gentle (and we want to)
    await asyncio.sleep(random.random() * 3 + 2)

with open('user_data.csv', 'w', newline='') as csvfile:
    # Create a CSV writer object
    writer = csv.DictWriter(csvfile, fieldnames=['ID', 'Username', 'First Name', 'Last Name'])
    # Write the header row
    writer.writeheader()

    # Iterate through each user in the dictionary and write data to the CSV
    for user_id, user_info in participants.items():
        # Extract relevant data
        user_dict = {
            'ID': user_id,
            'Username': user_info['username'],
            'First Name': user_info['first_name'],
            'Last Name': user_info['last_name']
        }
        # Write the user data row to the CSV
        writer.writerow(user_dict)

print("CSV file 'user_data.csv' created successfully!")

```

This code resulted in a CSV file and a dataframe that I've used to find more info on our participants. But that is a different and easier task.

Here is another useful piece of code:

```
userinfo = await client.get_entity('username')
print(userinfo)
```

This provides information on a user by ID or username. 

Example:

User(id=XXXXXXX, is_self=False, contact=False, mutual_contact=False, deleted=False, bot=False, bot_chat_history=False, bot_nochats=False, verified=False, restricted=False, min=False, bot_inline_geo=False, support=False, scam=False, apply_min_photo=True, fake=False, bot_attach_menu=False, premium=False, attach_menu_enabled=False, bot_can_edit=False, close_friend=False, stories_hidden=False, stories_unavailable=True, contact_require_premium=False, bot_business=False, access_hash=XXXXXXXX, first_name='XXX', last_name='XXXX', username='XXXXX', phone=None, photo=UserProfilePhoto(photo_id=XXXXXXXXX, dc_id=2, has_video=False, personal=False, stripped_thumb=b'XXXXXXXX), status=UserStatusRecently(by_me=False), bot_info_version=None, restriction_reason=[], bot_inline_placeholder=None, lang_code=None, emoji_status=None, usernames=[], stories_max_id=None, color=None, profile_color=None)

Sources
okhlopkov.com/how-to-get-a-telegram-channel-subscribers-list-in-python/





