# SourceCred Adept Camp Instance


See the cred: https://adept-camp.github.io/cred/#/explorer

Repos:

Bot: https://github.com/Adept-Camp/bot
Aracred: https://github.com/Adept-Camp/AraCred
This one :) Which is the SourceCred template instance for Adept Camp: https://github.com/Adept-Camp/cred/

This repository contains a template for running a SourceCred instance.

New users of SourceCred are encouraged to fork this repo to start their own
instance.

This repo comes with a GitHub action configured that will run SourceCred automatically
every 6 hours, as well as any time you change the configuration.

# About SourceCred Instances

SourceCred is organized around "instances". Every instance must have a
`sourcecred.json` file at root, which specifies which plugins are active in the
instance. Config and permanent data (e.g. the Ledger) are stored in the master branch.
All output or site data gets stored in the `gh-pages` branch by the Github Action.

Configuration files:
- `config/grain.json` specifies how much Grain to distribute every week. The `maxSimultaneousDistributions` parameter 
determines how many weeks of "back-distributions" to generate if the command hasn't been run in a while (or ever).
- `config/currencyDetails.json` By default, SourceCred uses "Grain" as the name for the currency it distributes to participants.
If you would like to change it to something custom for your community, you can do so by editing this file.
- `config/plugins/$PLUGIN_OWNER/$PLUGIN_NAME` stores plugin-specific data. Each
  plugin has its own directory. See the plugin section below to learn how to configure them.

Permanent Data:
- `data/ledger.json` keeps a history of all grain distributions and transfers as well as identities added / merged.

Generated Data: 

- `cache/` stores intermediate produced by the plugins. This directory should
  not be checked into Git at all.
- `output/` stores output data generated by SourceCred, including the Cred
  Graph and Cred Scores. This directory should be checked into Git; when
  needed, it may be removed and re-generated by SourceCred.
- `site/` which stores the compiled SourceCred frontend, which can display data
  stored in the instance.


# Setup and Usage

Using this instance as a starting point, you can update the config to include
the plugins you want, pointing at the data you care about. We recommend setting up
your instance locally first and make sure its working before pushing your changes
to master and using the Github Action.

Get [Yarn] and then run `yarn` to install SourceCred and dependencies.

Enable the plugins you want to use by updating the `sourcecred.json` file. e.g. 
to enable all the plugins:
```json
{
  "bundledPlugins": ["sourcecred/discourse", "sourcecred/discord", "sourcecred/github"]
}
```

Update the configuration files according to the plugin guides below.

Then, run the following commands to update the instance:

- `yarn load [...plugins]` loads the cache. By default, it loads all
  plugins, or it can load only specific plugins if requested.
- `yarn graph` regenerates plugin graphs from the cache;
  these graphs get saved in `output/`.
- `yarn score` computes Cred scores, combining data from all the chosen
  plugins
- `yarn grain` distributes Grain according to the current Cred scores, and the config in `config/grain.json`

**Generate the frontend:**

- `yarn site`

**Run the frontend:**

- `yarn serve`


If you want to clear the cached data, you can do so via:

- `yarn clean` 

Running `yarn clean` is a good idea if any plugins fail to load.

If you want to restart from a clean slate and remove all the generated graphs, you can do so via:

- `yarn clean-all` 

Run `yarn clean-all` if the `yarn graph` command fails due to a change in the config or breaking changes in a new version of SourceCred.
**Warning**: If you don't have credentials for every plugin, you might not be able to regenerate parts of the graph.

### Publishing on GitHub pages

Once you've got the instance configured to your satisfaction (see instructions on plugins below),
commit and push your changes to master (or make a pull request). The Github Action will then generate the frontend
and deploy it to GitHub Pages. To enable GitHub Pages for your instance, check out [this guide](https://docs.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).
Make sure you select `gh-pages` as the branch to publish from.



# Supported Plugins

## GitHub

The GitHub plugin loads GitHub repositories.

You can specify the repositories to load in
`config/plugins/sourcecred/github/config.json`.

The Github Action automatically has its own GITHUB_TOKEN, but if you need to load data from the 
GitHub plugin locally, you must have a GitHub API key in your environment as
`$SOURCECRED_GITHUB_TOKEN`. The key should be read-only without any special
permissions (unless you are loading a private GitHub repository, in which case
the key needs access to your private repositories).

You can generate a GitHub API key [here](https://github.com/settings/tokens).

## Discourse

The Discourse plugin loads Discourse forums; currently, only one forum can be loaded in any single instance. This does not require any special API
keys or permissions. You just need to set the server url in `config/plugins/sourcecred/discourse/config.json`.

## Discord

The Discord plugin loads Discord servers, and mints Cred on Discord reactions.
For instructions on configuring the Discord plugin, see the [Discord plugin page](https://sourcecred.io/docs/beta/plugins/discord/#configuration) in the SourceCred documentation. 

# Removing plugins

To deactivate a plugin, just remove it from the `bundledPlugins` array in the `sourcecred.json` file.
You can also remove its `config/plugins/OWNER/NAME` directory for good measure.

### Distributing Grain

This repo contains a GitHub action for distributing grain. It will run every Sunday and create a Pull Request
with the ledger updated with the new grain balances based on the users cred scores. The amount of grain to get distributed
every week can be defined in the `config/grain.json` file. There are two different policies that can be used to control
how the grain gets distributed: 
- `immediatePerWeek` splits the grain evenly based on everyone's Cred in the last week only.
- `balancedPerWeek` distributes the grain consistently based on total lifetime cred scores. i.e. it balances
the distribution of grain with the distribution of total historical cred.

The balanced policy allows SourceCred to reward people retro-actively. e.g. If someone has been historically "overpaid"
with grain relative to their cred scores, that people will receive less grain in the balanced distribution while people
who have been "underpaid" relative to their cred will receive more grain.

In SourceCred, we distribute 15000 grain / week with the "balanced" policy and 5000 grain / week with the "immediate"
policy. The values you use for your community depend on whether you want to optimize for more immediate short term
action, or for long term incentive alignment, but we recommend using a blend of both.

[Yarn]: https://classic.yarnpkg.com/

