## Accelerating PDEs using Latent Global Evolution


It introduces a simple, fast and scalable LE-PDE method to accelerate the simulation and inverse optimization of PDEs, which are crucial in many scientific and engineering applications (e.g., weather forecasting, material science, engine design).


# Installation

1. First clone the directory. Then run the following command to initialize the submodules:

```code
git submodule init; git submodule update
```

2. Install dependencies.

First, create a new environment using [conda](https://docs.conda.io/en/latest/miniconda.html) (with python >= 3.7). Then install pytorch, torch-geometric and other dependencies as follows (the repository is run with the following dependencies. Other version of torch-geometric or deepsnap may work but there is no guarentee.)

Install pytorch (replace "cu113" with appropriate cuda version. For example, cuda11.1 will use "cu111"):
```code
pip install torch==1.10.2+cu113 torchvision==0.11.3+cu113 torchaudio==0.10.2+cu113 -f https://download.pytorch.org/whl/torch_stable.html
```

Install torch-geometric. Run the following command:
```code
pip install torch-scatter==2.0.9 -f https://data.pyg.org/whl/torch-1.10.2+cu113.html
pip install torch-sparse==0.6.12 -f https://data.pyg.org/whl/torch-1.10.2+cu113.html
pip install torch-geometric==1.7.2
pip install torch-cluster==1.5.9 -f https://data.pyg.org/whl/torch-1.10.2+cu113.html
```

Install other dependencies:
```code
pip install -r requirements.txt
```

If you want to do inverse optimization, run also the following command inside the root directory of PDE_Control repository:
```code
pip install PDE-Control/PhiFlow/[gui] jupyterlab
```


# Dataset

The dataset files can be downloaded via [this link](https://drive.google.com/drive/folders/1rwcnT0g4_MiZfYUU4y7ybnfk8d4qgMEg?usp=share_link). 
* To run 1D experiment, download the files under "mppde1d_data/" in the link into the "data/mppde1d_data/" folder in the local repo. 
* To run 2D experiment, download the files under "fno_data/" in the link into the "data/fno_data/" folder in the local repo.
* To run inverse optimization experiment, download the files under "movinggas_data/" in the link into the "data/movinggas_data/" folder in the local repo.

# Training

Below we provide example commands for training LE-PDEs. For all the commands that reproduce the experiments in the paper, see the [results/README.md](https://github.com/snap-stanford/le_pde/blob/master/results/README.md).

An example 1D training command is:

```code
python train.py --exp_id=le-pde-1d --date_time=2022-11-21 --dataset=mppde1d-E2-50 --n_train=-1 --time_interval=1 --save_interval=2 --algo=contrast --no_latent_evo=False --encoder_type=cnn-s --input_steps=1 --evolution_type=mlp-3-elu-2 --decoder_type=cnn-tr --encoder_n_linear_layers=0 --n_conv_blocks=4 --n_latent_levs=1 --n_conv_layers_latent=3 --channel_mode=exp-32 --is_latent_flatten=True --evo_groups=1 --recons_coef=1 --consistency_coef=1 --contrastive_rel_coef=0 --hinge=0 --density_coef=0.001 --latent_noise_amp=1e-5 --normalization_type=gn --latent_size=128 --kernel_size=4 --stride=2 --padding=1 --padding_mode=zeros --act_name=elu --multi_step=1^2^3^4 --latent_multi_step=1^2^3^4 --temporal_bundle_steps=25 --use_grads=False --use_pos=False --is_y_diff=False --loss_type=rmse --loss_type_consistency=mse --batch_size=16 --val_batch_size=16 --epochs=50 --opt=adam --weight_decay=0 --seed=0 --gpuid=9 --id=0 --verbose=1 --save_iterations=1000 --latent_loss_normalize_mode=targetindi --n_workers=0 --static_encoder_type=param-0 --static_latent_size=3 --gpuid=0
```

An example 2D training command is:
```code
python train.py --exp_id=le-pde-2d --date_time=2022-11-21 --dataset=fno-4 --n_train=-1 --time_interval=1 --save_interval=10 --algo=contrast --no_latent_evo=False --encoder_type=cnn-s --input_steps=10 --evolution_type=mlp-3-elu-2 --decoder_type=cnn-tr --encoder_n_linear_layers=0 --n_conv_blocks=4 --n_latent_levs=1 --n_conv_layers_latent=3 --channel_mode=exp-16 --is_latent_flatten=True --evo_groups=1 --recons_coef=1 --consistency_coef=1 --contrastive_rel_coef=0 --hinge=0 --density_coef=0.001 --latent_noise_amp=1e-5 --normalization_type=gn --latent_size=256 --kernel_size=4 --stride=2 --padding=1 --padding_mode=zeros --act_name=elu --multi_step=1^2:0.1^3:0.1^4:0.1 --latent_multi_step=1^2^3^4 --use_grads=False --use_pos=False --is_y_diff=False --loss_type=mse --loss_type_consistency=mse --batch_size=20 --val_batch_size=20 --epochs=200 --opt=adam --weight_decay=0 --seed=0 --gpuid=9 --id=0 --verbose=1 --save_iterations=400 --latent_loss_normalize_mode=targetindi --n_workers=0 --gpuid=0
```

An example command of training for smoke simulation (used for inverse design) is:
```code
python train.py --exp_id=le-pde-smoke --date_time=2022-11-21 --dataset=movinggas --n_train=-1 --time_interval=1 --save_interval=10 --algo=contrast --reg_type=None --reg_coef=0 --is_reg_anneal=True --no_latent_evo=False --encoder_type=cnn-s --evolution_type=mlp-3-elu-2 --decoder_type=cnn-tr --encoder_n_linear_layers=0 --n_conv_blocks=4 --n_latent_levs=1 --n_conv_layers_latent=3 --channel_mode=exp-16 --is_latent_flatten=True --evo_groups=1 --recons_coef=1 --consistency_coef=1 --contrastive_rel_coef=0 --hinge=0 --density_coef=0.001 --latent_noise_amp=1e-5 --normalization_type=gn --latent_size=128 --kernel_size=4 --stride=2 --padding=1 --padding_mode=zeros --act_name=elu --multi_step=1^2:0.1^3:0.1^4:0.1 --latent_multi_step=1^2^3^4 --use_grads=False --use_pos=False --is_y_diff=False --loss_type=mse --loss_type_consistency=mse --batch_size=16 --val_batch_size=16 --epochs=100 --opt=adam --weight_decay=0 --seed=0 --id=0 --verbose=1 --save_iterations=1000 --latent_loss_normalize_mode=targetindi --n_workers=0 --static_encoder_type="cnn-s" --static_latent_size=16 --gpuid=0
```

The results are saved under `results/{--exp_id}_{--date_time}/` (here `--exp_id` and `--date_time` are according to the command for training). Each experiment file has the following suffix: "*Hash_{hash}_{machine-name}.p". The hash (e.g., "Un6ae7ja"), is uniquely generated according to all the configurations of the argparse (if any argument is different, it will result in a different hash).


