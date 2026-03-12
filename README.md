# SpotUI.nvim

Because alt-tabbing to Spotify is for people who don't use Neovim.

![expanded-ui](assets/full-ss.png)

## What it does

When you toggle it open, a window pops up in your chosen corner with the album
art and current track. After a few seconds it shrinks down to a compact bar
showing the song, artist, and timestamp. It expands again automatically on every
song change.

![mini-ui](assets/mini-ss.png)

## Requirements

- Neovim 0.8+
- [chafa](https://hpjansson.org/chafa/) — for album art rendering
- curl — for Spotify API calls
- A Spotify account

**Installing chafa:**
```bash
# macOS
brew install chafa

# Ubuntu/Debian
sudo apt install chafa

# Windows
scoop install chafa
```

## Setup

**1. Create a Spotify app**

Go to [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard),
create a new app, and set the redirect URI to `http://127.0.0.1:8888/callback`.
Grab your Client ID and Client Secret.

**2. Get your tokens**

Create a `.env` file in the project root:
```
SPOTIFY_CLIENT_ID=your_client_id
SPOTIFY_CLIENT_SECRET=your_client_secret
```

Then run the setup script once:
```bash
pip install requests python-dotenv
python scripts/get_token.py
```

This opens a browser window, asks you to authorize the app, and saves a token
file to `~/.spotify_nvim_tokens.json`. You only need to do this once.

**3. Install the plugin**

With [lazy.nvim](https://github.com/folke/lazy.nvim):
```lua
{
  'AaravB23/spotui-nvim',
  config = function()
    require('spotui').setup({
      position = 'bottom-left',
      poll_interval = 2000,
      window = {
        width = 30,
        expand_duration = 1500,
      }
    })
  end
}
```

## Keymaps

| Key | Action |
|-----|--------|
| `<leader>sp` | Toggle the window |
| `<leader>ss` | Play / pause |
| `<leader>sn` | Next track |
| `<leader>sb` | Previous track |

## Configuration
These are the defaults:
```lua
require('spotui').setup({
  poll_interval = 2000,      -- how often to check Spotify (ms)
  position = 'bottom-left',  -- 'top-right', 'top-left', 'bottom-right', 'bottom-left'
  window = {
    width = 30,
    expanded_height = 16,    -- height when album art is visible
    compact_height = 3,      -- height after minimizing
    expand_duration = 1500,  -- ms before shrinking to compact view
  },
  highlights = {
    background = 'NormalFloat',  -- window background highlight group
    border = 'FloatBorder',      -- border highlight group
    text = 'NormalFloat',        -- text highlight group
  }
})
```

To blend the window into your colorscheme instead of using the default float colors:
```lua
highlights = {
  background = 'Normal',
  border = 'Normal',
  text = 'Normal',
}
```

Any valid Neovim highlight group works here. Run `:Telescope highlights` or
`:highlight` to browse what's available in your current theme.


## How it works

SpotUI polls the Spotify Web API every `poll_interval` milliseconds using async
curl calls via `vim.loop.spawn` so it never blocks the editor. Album art is
downloaded once per track and cached for the session, then rendered as Unicode
block characters using chafa.

## Notes

- Album art rendering requires a terminal with Unicode support (most modern
  terminals work fine eg. Windows Terminal, WezTerm, Kitty, iTerm2, Alacritty)
- Playback controls require Spotify Premium
- The token file is stored at `~/.spotify_nvim_tokens.json` and refreshes
  automatically when it expires

## Roadmap / To-do

- [ ] Playlist switcher: fuzzy finder for recent playlists via Telescope
- [ ] Statusline component: expose a function for lualine/heirline integration
- [ ] Colored album art: parse ANSI codes into Neovim highlight groups
- [ ] Linux/macOS testing: currently developed on Windows, needs validation on other platforms
