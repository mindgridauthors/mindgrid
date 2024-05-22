# MindGrid

We present MindGrid, a research toolkit for studying practical alignment in human-AI collaboration tasks via assistive communication processes.

## Installation
Clone the repository.
```
git clone https://github.com/mindgridauthors/mindgrid.git
```
Create a new conda environment and install dependencies.
```
conda create -n mindgrid python=3.10
conda activate mindgrid
cd mindgrid
pip install -r requirements.txt
```

## Quickstart Example
Quickly play around with all that MindGrid has to offer with this quickstart example. Make sure you have defined a config YAML file, say `config.yaml`. Check out `mindgrid/configs/base.yaml` for inspiration!
```
from mindgrid.builder import make_env
from mindgrid.ifnrastructure.config_utils import make_config

with open("/path/to/config.yaml", "r") as f:
    config_str = f.read()

game_config = make_config(config_str = config_str)

env = make_env(getattr(game_config, game_config.roles.executor).world_model)

env.render_mode = "human"

env.render()
```

## Datasets
Inside `datasets/` are the four {environment mismatch, skillset mismatch} x {games, datapoints} datasets, complete with five training, validation, and testing splits. Unzip the `.zip` files to get the corresponding `pickle` files. Once you have them, here is a quick example that shows how to load them and what they contain.
```
>>> import pickle
>>> with open("datasets/skillset_mismatch_datapoints.pickle", "rb") as f:
...     skillset_datapoints = pickle.load(f)
... 
>>> skillset_datapoints.keys()
dict_keys(['train', 'val_out', 'test_out', 'test_in', 'val_in'])
>>> skillset_datapoints["train"][0].keys()
dict_keys(['game_id', 'skill_name', 'arguments', 'instruction', 'states', 'full_obs', 'full_text_obs', 'partial_obs', 'partial_text_obs', 'actions', 'id'])
>>> with open("datasets/skillset_mismatch_games.pickle", "rb") as f:
...     skillset_games = pickle.load(f)
... 
>>> skillset_games.keys()
dict_keys(['train', 'val_out', 'test_out', 'test_in', 'val_in'])
>>> skillset_games["train"][0].keys()
dict_keys(['config', 'ref_plan', 'id'])
>>> with open("datasets/environment_mismatch_datapoints.pickle", "rb") as f:
...     environment_datapoints = pickle.load(f)
... 
>>> environment_datapoints.keys()
dict_keys(['train', 'val_out', 'test_out', 'test_in', 'val_in'])
>>> environment_datapoints["train"][0].keys()
dict_keys(['game_id', 'init_description', 'edits', 'edit_descriptions', 'actions', 'states', 'full_obs', 'full_text_obs', 'partial_obs', 'partial_text_obs', 'queries', 'id'])
>>> with open("datasets/environment_mismatch_games.pickle", "rb") as f:
...     environment_games = pickle.load(f)
... 
>>> environment_games.keys()
dict_keys(['train', 'val_out', 'test_out', 'test_in', 'val_in'])
>>> environment_games["train"][0].keys()
dict_keys(['config', 'ref_plan', 'id'])
```
Instead of making your own config YAML file, you can load one directly from one of the datasets as follows:
```
config_str = skillset_games["train"][0]["config"]
game_config = make_config(config_str = config_str)
# Rest of the quickstart example is the same
```

## Recreating Experiments
Our experiments—and other LLM-based agent operations in MindGrid—require the use of LLM APIs. Before running any LLM-based code, create a file called `access_tokens.py` inside `mindgrid` to store your API keys. The file should look something like this:
```
OPENAI_KEY = "yourkeyhere"
SCALE_KEY = "yourkeyhere"
HUGGINGFACE_KEY = "yourkeyhere"
```

To recreate, for example, the LLM prompting experiment in the paper for the environment mismatch speaker task, using few-shot prompting with LLaMA 3 70B Instruct, run the following:
```
python3 baselines/prompted_llm/belief_baseline.py -s -id -m 0 -f
```
`belief_baseline.py` contains code for the environment mismatch experiments. `intention_baseline.py` in the same directory contains code for the skillset mismatch experiments.

Flags:
- `-s`: run the speaker task experiment
- `-l`: run the listener task experiment
- `-id`: use the in-distribution test set
- `-ood`: use the out-of-distribution test set
- `-m`: the model index to use. We set `0` to LLaMA 3 70B Instruct, `1` to Mixtral 8x7B Instruct, and `2` to Gemma 7B Instruct
- `-f`: use few-shot prompting (without this flag, zero-shot prompting is used)

Model outputs will be logged inside the respective `few_shot/` and `zero_shot/` directories in `prompted_llm/`.

To evaluate model performance, run something like
```
python3 baselines/prompted_llm/evaluate.py -b -s
```
This will automatically impute the scores into `scores.csv` and output images as well inside `prompted_llm/`.

Flags:
- `-b`: evaluate environment mismatch
- `-i`: evaluate skillset mismatch
- `-s`: evaluate the speaker task experiment
- `-l`: evaluate the listener task experiment
- `-o`: get an overview of average scores per (mismatch, task, model). This requires a complete `scores.csv`
