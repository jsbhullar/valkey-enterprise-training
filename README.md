
## ⚙️ Prerequisites

To get the most out of this training and run the code examples, you'll need the following installed on your system:

* [**Git**](https://git-scm.com/downloads): For cloning the repository.
* [**Docker**](https://www.docker.com/get-started): To easily run a Valkey server without local installation.
* [**Docker Compose**](https://docs.docker.com/compose/install/): For managing the Valkey container.
* [**Python 3.8+**](https://www.python.org/downloads/): The programming language used for all code examples.
* [**pip**](https://pip.pypa.io/en/stable/installation/): Python's package installer (usually comes with Python).

## 🚀 Getting Started

Follow these steps to set up your environment and start exploring the training materials:

1. **Clone the Repository:**
   
   ```bash
   git clone https://github.com/jsbhullar/valkey-enterprise-training.git
   cd your-repo-name
   ```
   
   

2. **Start the Valkey Server:**
   Navigate to the root of the cloned repository and use Docker Compose to spin up a Valkey instance:
   
   ```bash
   docker compose up -d
   ```
   
   This command will download the `valkey/valkey` image (if not already present) and start a Valkey server listening on port `6379`.

3. **Explore the Concepts:**
   Head over to the `concepts/` directory to start with the theoretical explanations. It's recommended to follow them in order:
   
   * [01 - Introduction to Valkey](concepts/01-introduction-to-valkey.md)
   * [02 - Core Data Structures](concepts/02-core-data-structures.md)
   * [03 - Caching Strategies](concepts/03-caching-strategies.md)
   * [04 - Publish/Subscribe](concepts/04-pub-sub.md)
   * [05 - Message Queues](concepts/05-message-queues.md)

4. **Run Code Examples (Python):**
   Each concept file may refer to corresponding code examples. Navigate to the relevant example folder within `code-examples/python/`.
   
   For example, to run the caching strategies example:
   
   ```bash
   cd code-examples/python/03-caching-strategies
   # Install Python dependencies for this specific example
   pip install -r requirements.txt
   # Run the example script
   python main.py
   ```
   
   _Please refer to the `README.md` within each specific example folder for detailed instructions on how to run that particular demonstration._

5. **Stop the Valkey Server:**
   Once you're done with your session, you can stop and remove the Valkey container:
   
   ```bash
   docker compose down
   ```

## 🙏 Contributing

We welcome contributions to this training! Whether it's correcting typos, improving explanations, adding new code examples, or proposing entirely new concepts, your help is highly appreciated.

Please refer to the [CONTRIBUTING.md](contributing.md) file for detailed guidelines on how to contribute.

## 📄 License

This project is licensed under the [MIT License](LICENSE) - see the `LICENSE` file for details.

---

_Happy Learning!_ 🚀

```

---
