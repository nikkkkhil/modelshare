<h2 align="center"> modelshare - A flexible tool for building and sharing deep learning modules </h2>


modelshare is a flexible deep learning module manager, which aims at encouraging code reuse and sharing. It ships with a bunch of useful features, such as CLI based module management, runtime checking,  and experimental task runner, etc.  You can integrate modelshare with PyTorch, Tensorflow, MXNet, or any deep learning framework you like that provides a python interface. 

Moreover, a set of [Pytorch-backend modelshare modules](https://github.com/nikkkkhil/modelshare/tree/pytorch), e.g., network trainer, data loader, optimizer, dataset, visdom logging, are already provided. More modules and framework support will be added later. 

---

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
    - [Create your first modelshare module](#create-your-first-modelshare-module)
    - [Use your modelshare module in Python](#use-your-modelshare-module-in-python)
    - [Debug your modelshare modules](#debug-your-modelshare-modules)
    - [Install modelshare modules from local path](#install-modelshare-modules-from-local-path)
    - [Install modelshare modules from URL](#install-modelshare-modules-from-url)
    - [Uninstall modelshare modules](#uninstall-modelshare-modules)
    - [Version control modelshare modules](#version-control-modelshare-modules)
    - [Use modelshare to manage your experiments](#use-modelshare-to-manage-your-experiments)
- [Contact](#contact)
- [Issues](#issues)
- [Contribution](#contribution)
- [License](#license)

## Prerequisites
* System (tested on Ubuntu 14.04LTS, Win10, and MacOS *High Sierra*)
* [Python](https://www.python.org) >= 3.5.4
* [Git](https://git-scm.com)

## Installation
```bash

# manually download and install
git clone https://github.com/nikkkkhil/modelshare.git
pip install ./modelshare
```

## Basic Usage
> The official website and documentation are under construction.

### Create your first modelshare module
1. Create "hello.py" under your current path with the following content:

    ```python
    from modelshare import register

    @register(author='nikkkkhil', version='1.0.0')
    def hello_modelshare(name: str) -> str:
        """My first modelshare module!"""

        return 'Hello ' + name
    ```

    > Note that the type of module parameters and return values must be clearly defined. This helps the user to better understand the module, and at runtime modelshare automatically checks whether each module receives and outputs as expected, thus helping you to identify potential bugs earlier.

2. Execute the following command in your shell to verify the module:

    ```bash
    $ modelshare module list -v
    # Output:
    # 
    # 1 modelshare module found.
    # [0] main.hello_modelshare (1.0.0) by "nikkkkhil":
    #     hello_modelshare(
    #         name:str) -> str

    # Documentation:
    #     My first modelshare module!
    #     author: nikkkkhil
    #     module_path: /Users/nikkkkhil/Workspace/modelshare.doc
    #     version: 1.0.0 
    ```

    > Note that all modules under current path are registered under the "**main**" namespace.

    > With the CLI tool, you can easily manage modelshare modules. Execute `modelshare -h` for more details. 

3. That's it. You just created a simple modelshare module!


### Use your modelshare module in Python
1. Open an interactive python interpreter under the same path of "hello.py" and run following commands:

    ```python
    >>> from modelshare import modules
    >>> print(modules.hello_modelshare) # access the module
    # Output:
    # 
    # hello_modelshare(
    # name:str) -> str
    >>> print(modules['*_nes?']) # wildcard search
    # Output:
    # 
    # hello_modelshare(
    # name:str) -> str
    >>> print(modules['r/main.\w+_modelshare']) # regex search
    # Output:
    # 
    # hello_modelshare(
    # name:str) -> str
    >>> modules.hello_modelshare('nikkkkhil') # use the module
    # Output:
    #
    # 'Hello nikkkkhil'
    >>> modules.hello_modelshare(123) # runtime type checking
    # Output:
    #
    # TypeError: The param "name" of modelshare module "hello_modelshare" should be type of "str". Got "123".
    >>> modules.hello_modelshare('nikkkkhil', wrong=True)
    # Output:
    #
    # Unexpected param(s) "wrong" for modelshare module:
    # hello_modelshare(
    # name:str) -> str
    ```

    > Note that modelshare automatically imports modules and checks them as they are used to make sure everything is as expected.

2. You can also directly import modules like this:

    ```python
    >>> from modelshare.main.hello import hello_modelshare
    >>> hello_modelshare('World')
    # Output:
    #
    # 'Hello World'
    ```

    > The import syntax is `from modelshare.<namespace>.<filename> import <module_name>`

3. Access to modelshare modules through code is flexible and easy.

### Debug your modelshare modules
1. Open an interactive python interpreter under the same path of "hello.py" and run following commands:

    ```python
    >>> from modelshare import modules
    >>> modules.hello_modelshare('nikkkkhil')
    # Output:
    #
    # 'Hello nikkkkhil'
    ```

2. Keep the interpreter **OPEN** and use an externel editor to modify the "hello.py":

    ```python
    # change Line7 from "return 'Hello ' + name" to
    return 'Nice to meet you, ' + name
    ```

3. Back to the interpreter and rerun the same command:

    ```python
    >>> modules.hello_modelshare('nikkkkhil')
    # Output:
    #
    # 'Nice to meet you, nikkkkhil'
    ```

    > Note that modelshare detects source file modifications and automatically reloads the module.

4. You can use this feature to develop and debug your modelshare modules efficiently.

### Install modelshare modules from local path
1. Create a folder `my_namespace` and move the `hello.py` into it:

    ```bash
    $ mkdir my_namespace
    $ mv hello.py ./my_namespace/
    ```

2. Create a new file `more.py` under the folder `my_namespace` with the following content:

    ```python
    from modelshare import register

    @register(author='nikkkkhil', version='1.0.0')
    def sum(a: int, b: int) -> int:
        """Sum two numbers."""

        return a + b

    # There is no need to repeatedly declare meta information
    # as modules within the same file automatically reuse the 
    # previous information. But overriding is also supported.
    @register(version='2.0.0')
    def mul(a: float, b: float) -> float:
        """Multiply two numbers."""
        
        return a * b
    ```

    > Now we have:
    ```
    current path/
    ├── my_namespace/
    │   ├── hello.py
    │   ├── more.py
    ```

3. Run the following command in the shell:

    ```bash
    $ modelshare module install ./my_namespace hello_word
    # Output:
    #
    # Install "./my_namespace/" -> Search paths. Continue? (Y/n) [Press <Enter>]
    ```

    > This command will add  "**my_namespace**" folder to modelshare's search path, and register all modelshare modules in it under the namespace "**hello_word**". If the last argument is omitted, the directory name, "my_namespace" in this case, is used as the namespace.

4. Verify the installation via CLI:

    ```bash
    $ modelshare module list
    # Output:
    #
    # 3 modelshare modules found.
    # [0] hello_world.hello_modelshare (1.0.0)
    # [1] hello_world.mul (2.0.0)
    # [2] hello_world.sum (1.0.0)
    ```

    > Note that those modelshare modules can now be accessed regardless of your working path.

5. Verify the installation via Python interpreter:

    ```bash
    $ ipython # open IPython interpreter
    ```
    ```python
    >>> from modelshare import modules
    >>> print(len(modules))
    # Output:
    #
    # 3
    >>> modules.[Press <Tab>] # IPython Auto-completion
    # Output:
    #
    # hello_modelshare
    # mul
    # sum
    >>> modules.sum(3, 2)
    # Output:
    #
    # 5
    >>> modules.mul(2.5, 4.0)
    # Output:
    #
    # 10.0
    ```

6. Thanks to the auto-import feature of modelshare, you can easily share modules between different local projects.
    
### Install modelshare modules from URL
1. You can use the CLI tool to install modules from URL:

    ```bash
    # select one of the following commands to execute
    # 0. install from Github repo via short URL (GitLab, Bitbucket are also supported)
    $ modelshare module install github@nikkkkhil/modelshare:pytorch pytorch
    # 1. install from Git repo
    $ modelshare module install "-b pytorch https://github.com/nikkkkhil/modelshare.git" pytorch
    # 2. install from zip file URL
    $ modelshare module install "https://github.com/nikkkkhil/modelshare/archive/pytorch.zip" pytorch
    ```

    > The last optional argument is used to specify the namespace, "**pytorch**" in this case.

2. Verify the installation:

    ```bash
    $ modelshare module list
    # Output:
    #
    # 26 modelshare modules found.
    # [0] hello_world.hello_modelshare (1.0.0)
    # [1] hello_world.mul (2.0.0)
    # [2] hello_world.sum (1.0.0)
    # [3] pytorch.adadelta_optimizer (0.1.0)
    # [4] pytorch.checkpoint (0.1.0)
    # [5] pytorch.cross_entropy_loss (0.1.0)
    # [6] pytorch.fetch_data (0.1.0)
    # [7] pytorch.finetune (0.1.0)
    # [8] pytorch.image_transform (0.1.0)
    # ...
    ```

### Uninstall modelshare modules
1. You can remove modules from modelshare's search path by executing:

    ```bash
    # given namespace
    $ modelshare module remove hello_world
    # given path to the namespace
    $ modelshare module remove ./my_namespace/
    ```

2. You can also delete the corresponding files by appending a `--delete` or `-d` flag:

    ```bash
    $ modelshare module remove hello_world --delete
    ```

### Version control modelshare modules

1. When installing modules, modelshare adds the namespace to its search path without modifying or moving the original files. So you can use any version control system you like, e.g., Git, to manage modules. For example:

    ```bash
    $ cd <path of the namespace>
    # update modules
    $ git pull
    # specify version
    $ git checkout v1.0
    ```

2. When developing a modelshare module, it is recommended to define meta information for the module, such as the author, version, requirements, etc. Those information will be used by modelshare's CLI tool. There are two ways to set meta information:

    * define meta information in code

    ```python
    from modelshare import register

    @register(author='nikkkkhil', version='1.0')
    def my_module() -> None:
        """My Module"""
        pass
    ```

    * define meta information in a `modelshare.yml` under the path of namespace

    ```YAML
    author: nikkkkhil
    version: 1.0
    requirements:
        - {url: opencv, tool: conda}
        # default tool is pip
        - torch>=0.4
    ```

    > Note that you can use both ways at the same time.

### Use modelshare to manage your experiments
1. Make sure you have Pytorch-backend modules installed, and if not, execute the following command:

    ```bash
    $ modelshare module install github@nikkkkhil/modelshare:pytorch pytorch
    ```

2. Create "**train_mnist.yml**" with the following content:

    ```YAML
    _name: network_trainer
    data_loaders:
      _name: fetch_data
      dataset: 
        _name: mnist
        data_dir: ./data
      batch_size: 128
      num_workers: 4
      transform:
        _name: image_transform
        image_size: 28
        mean: [0.1307]
        std: [0.3081]
      train_splits: [train]
      test_splits: [test]
    model:
      _name: lenet5
    criterion:
      _name: cross_entropy_loss
    optimizer:
      _name: adadelta_optimizer
    meters:
      top1:
        _name: topk_meter
        k: 1
    max_epoch: 10
    device: cpu
    hooks:
      on_end_epoch: 
        - 
          _name: print_state
          formats:
            - 'epoch: {epoch_idx}'
            - 'train_acc: {metrics[train_top1]:.1f}%'
            - 'test_acc: {metrics[test_top1]:.1f}%'   
    ```

    > Check [HERE](https://github.com/nikkkkhil/modelshare/tree/pytorch/demo) for more comprehensive demos.

3. Run your experiments through CLI:

    ```bash
    $ modelshare task run ./train_mnist.yml
    ```

4. You can also use modelshare's task runner in your code:

    ```python
    >>> from modelshare import run_tasks
    >>> run_tasks('./train_mnist.yml')
    ```

5. Based on the task runner feature, modelshare modules can be flexibly replaced and assembled to create your desired experiment settings.

## Contact 
nikkkkhil  

## Issues
Feel free to submit bug reports and feature requests.

## Contribution
Pull requests are welcome.

## License
[MIT](https://opensource.org/licenses/MIT)

Copyright © 2018-present, nikkkkhil 
