# Cosmos-based Node Ansible Setup Plus Several Supporting Playbooks

## Design Philosophy

1. Extendable to most Tendermint-based chains
1. Support both mainnet and testnet
1. Stable playbooks and roles; Customizable variables
1. Support essential functions (snapshot, state-sync, public RPC/API endpoints and Cosmos Exporter) through separate playbooks

## TL/DR

You run one playbook and set up a node.

```bash
ansible-playbook main.yml -e "target=juno_main"
```

Because we try our best to support the latest node version, it is not recommended for you to sync from Block 1. Rather, please [state-sync](https://polkachu.com/state_sync) or start from a [snapshot](https://polkachu.com/tendermint_snapshots).

## Node deployment (Validator, Backup and Relayer)

For every network where we run a validator on mainnet, we run 3 nodes (Validator, Backup and Relayer). The details of our 3-node infrastructure are documented [here](https://polkachu.com/blogs/holy-trinity-a-system-approach-to-tendermint-based-chain-validation).

#### Opinionated Configuration

We have 2 strong opinions about the node configuration:

1. Each network will have its custom port prefix. This is to prevent port collision if you run multiple nodes on the same server (we do so for Backup Node and Relayer Node). For example, Juno's custom port prefix is 126 and that of Osmosis is 125. Since it is rather arbitrary, we are going to force the same convention on you unless you fork the code.
1. Each type of node will have its setting based on Polkachu's "best practice". For example, the main node (Validator) has null indexer, and 100/0/<prime number> pruning, and Relayer node has kv indexer and 40000/2000/<prime number> pruning. We will force these setting on you unless you fork the code.

#### Host Variables

Take a look at the `inventory.sample` file. You will see an example `juno` group with 3 different hosts: `juno_main`, `juno_backup`, and `juno_relayer`. Each host will have the following variables:

1. `ansible_host`: Required. The IP address of the server.
1. `type`: Required. It can be `main`, `backup` and `relayer` (also `test` if you are adventurous). Each is opinionated in its configuration settings.
1. `prepare`: Optional. If unset, it is default to true. If `false`, it will skip setups of firewall, go, cosmovisor, node exporter, promtail, etc. The reason for the `false` option is because we run many backup/relayer nodes on the same server with setup done already.

#### Other Variables

Besides the above host variables, you will also specify the following `all` variables in the inventory file:

1. `ansible_user`: The sample file assumes `ubuntu`, but feel free to use other user name. This user need sudo privilege.
1. `ansible_port`: The sample file assumes `22`. But if you are like me, you will have a different ssh port other than `22` to avoid port sniffing.
1. `ansible_ssh_private_key_file`: The sample file assumes `~/.ssh/id_rsa`, but you might have a different key location.
1. `var_file`: It tells the program where to look for the variable file. This is useless for the mainnet, because the var file will automatically be inferred by the network name. However, it is essentially for testnets.
1. `user_dir`: The user's home directory. In the sample inventory file this is a computed variable based on the ansible_user. It assumes that it is not a root user and its home directory is `/home/{{ansible_user}}`.
1. `path`: This is to make sure that the ansible_user can access the `go` executable.
1. `node_exporter`: Default is `true`. Change it to `false` if you do not want to install node_exporter
1. `promtail`: Default is `true`. Change it to `false` if you do not want to install promtail
1. `log_monitor`: Enter your monitor server IP if you install promtail.
1. `node_name`: This is your node name for the config.toml file.
1. `log_name`: This is the server name for the promtail service.

#### Ready? Go!

One you understand the setup, please first copy it to your own inventory file so you can customize it to suit your needs:

```bash
cp inventory.sample inventory
```

When you are ready install a node, you run:

```bash
ansible-playbook main.yml -e "target=HOST_NAME"
```

## Playbooks

| Playbook                        | Description                                                                                      |
| ------------------------------- | ------------------------------------------------------------------------------------------------ |
| `main.yml`                      | The main playbook to set up a node                                                               |
| `prepare.yml`                   | Prepare the server with node exporter, promtail, go, cosmovisor, and firewall rules              |
| `support_cosmos_exporter.yml `  | Set up Cosmos Exporter configuration (assuming Cosmos Exporter already installed)                |
| `support_public_endpoints.yml ` | Set up Nginx reverse proxy for public PRC/ API                                                   |
| `support_snapshot.yml `         | Install snapshot script and a cron job                                                           |
| `support_state_sync.yml `       | Install state-sync script                                                                        |
| `support_seed.yml `             | Install seed node with Tenderseed. You need a node_key.json.j2 file so the node_id is consistent |
| `system_update.yml `            | Update a server and restart if needed                                                            |

### Playbook Usage Example

```bash
ansible-playbook support_seed.yml -e "target=umee_seed seed=190c4496f3b46d339306182fe6a507d5487eacb5@65.108.131.174:36656"
```

## Supported Networks

| Network       | Mainnet | Testnet | Port Prefix |
| ------------- | ------- | ------- | ----------- |
| Agoric        | Yes     | Yes     | 144         |
| Akash         | Yes     |         | 128         |
| Althea        |         | Yes     | 124         |
| Archaway      |         | Yes     | 115         |
| Assetmantle   | Yes     |         | 146         |
| Axelar        | Yes     | Yes     | 151         |
| Bitcanna      | Yes     |         | 130         |
| Bitsong       | Yes     |         | 160         |
| Canto         | Yes     |         | 155         |
| Celestia      |         | Yes     | 116         |
| Cerberus      | Yes     | Yes     | 138         |
| Certik        | Yes     |         | 140         |
| Cheqd         | Yes     |         | 161         |
| Chihuahua     | Yes     | Yes     | 129         |
| Comdex        | Yes     | Yes     | 131         |
| Cosmos        | Yes     |         | 149         |
| Craft         |         | Yes     | 157         |
| Crescent      | Yes     |         | 145         |
| Cudos         | Yes     |         | 123         |
| Defund        |         | Yes     | 112         |
| Desmos        | Yes     |         | 162         |
| Deweb         |         | Yes     | 114         |
| DIG           | Yes     |         | 163         |
| Echelon       | Yes     |         | 120         |
| Evmos         | Yes     | Yes     | 134         |
| Fetch         | Yes     |         | 152         |
| Firmachain    | Yes     |         | 164         |
| Galaxy        | Yes     |         | 148         |
| Gitopia       |         | Yes     | 113         |
| Gravity       | Yes     |         | 142         |
| Hypersign     |         | Yes     | 109         |
| IDEP          | Yes     |         | 165         |
| Impacthub     | Yes     |         | 166         |
| Injective     | Yes     |         | 143         |
| Juno          | Yes     | Yes     | 126         |
| Kava          | Yes     |         | 139         |
| Kichain       | Yes     | Yes     | 135         |
| Konstellation | Yes     |         | 133         |
| Kujira        | Yes     | Yes     | 118         |
| Kyve          |         | Yes     | 110         |
| Lum           | Yes     |         | 167         |
| Meme          | Yes     | Yes     | 147         |
| Nym           | Yes     |         | 153         |
| Odin          | Yes     |         | 168         |
| Omniflix      | Yes     |         | 169         |
| Osmosis       | Yes     |         | 125         |
| Paloma        |         | Yes     | 121         |
| Passage       | Yes     |         | 156         |
| Quicksilver   |         | Yes     | 111         |
| Secret        |         |         | 171         |
| Sifchain      | Yes     |         | 132         |
| Sommelier     | Yes     |         | 141         |
| Sei           |         | Yes     | 119         |
| Source        |         | Yes     | 158         |
| Stargaze      | Yes     | Yes     | 137         |
| Stride        | Yes     | Yes     | 122         |
| Teritori      |         | Yes     | 159         |
| Terra2        | Yes     | Yes     | 117         |
| Umee          | Yes     | Yes     | 136         |
| Vidulum       | Yes     |         | 170         |

## V1 to V2 migration [OPTIONAL]

In V1, the custom port prefix is 2 digits. However, this hobby project has evolved into a more ambitious one and we have run out of the prefixes. Therefore, V2 introduces a breaking change of the 3-digit custom port prefixes.

If you have a node running based on V1 port prefix system, you do not need to do anything. However, if you are as OCD as Polkachu, you might want to migrate all the previous nodes to comply with the new system. Here is a playbook to manage the migration. You still need to close the old ports that are not longer in use, but this playbook should take care of the rest.

```bash
ansible-playbook support_config_update.yml -e "target=juno_main"
```

## Known Issue

Because this repo tries to accommodate as many Tendermint-based chains as possible, it cannot adapt to all edge cases. Here are some known issues and how to resolve them.

| Chain            | Issue                                                    | Solution                                                                             |
| ---------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Axelar           | Some extra lines at the end of app.toml                  | Delete extra lines and adjust some settings these extra lines are supposed to change |
| Canto            | genesis file needs to be unwrapped from .result.genesis  | Unwrap genesis with jq command                                                       |
| Injective        | Some extra lines at the end of app.toml                  | Delete extra lines and adjust some settings these extra lines are supposed to change |
| Kichain          | Some extra lines at the end of app.toml                  | Delete extra lines and adjust some settings these extra lines are supposed to change |
| Celestia testnet | inconsistent config.toml file variable naming convention | Manually adjust config.toml file                                                     |
