# 🤝 Contributing to Valkey for Enterprise Use Cases Training

Thank you for your interest in contributing to this Valkey training repository! Your contributions help make this resource more accurate, comprehensive, and valuable for everyone.

This document outlines the guidelines for contributing to this project. By participating, you agree to abide by our Code of Conduct.

## ✨ Code of Conduct

Please note that this project is released with a Contributor Code of Conduct. By participating in this project, you agree to abide by its terms. You can find the full text of the [Contributor Covenant here](https://www.contributor-covenant.org/version/2/0/code_of_conduct.html).

## 💡 What We're Looking For

We welcome contributions of all kinds, including but not limited to:

* **Bug Reports:** If you find errors in the concepts, code, or documentation.
* **Fixes:** Correcting typos, grammatical errors, broken links, or incorrect code.
* **Improvements:** Clarifying explanations, refining existing code examples, or enhancing the overall readability.
* **New Code Examples:** Demonstrations of Valkey features or use cases not yet covered, following the existing Python structure.
* **New Concepts:** Proposing and, if you're up for it, developing new modules/`.md` files for enterprise-relevant Valkey topics (e.g., advanced security, specific module usage, performance tuning).
* **Updates:** Keeping the content current with the latest Valkey versions or best practices.

## 🚀 How to Contribute

The general workflow for contributing is as follows:

1. **Fork the Repository:** Click the "Fork" button at the top right of this repository's page on GitHub. This creates a copy of the repository under your GitHub account.

2. **Clone Your Fork:** Clone your forked repository to your local machine:
   
   ```bash
   git clone https://github.com/YOUR_USERNAME/valkey-enterprise-training.git
   cd valkey-enterprise-training
   ```
   
   (Replace `YOUR_USERNAME` with your GitHub username).

3. **Create a New Branch:** Before making any changes, create a new branch for your specific contribution. Use a descriptive name:
   
   ```bash
   git checkout -b feature/my-new-example
   # or
   git checkout -b fix/typo-in-readme
   ```

4. **Make Your Changes:**
   
   * Edit existing files or add new ones.
   * For code examples, ensure they are in the `code-examples/python/` directory, ideally in a new subfolder (e.g., `code-examples/python/06-valkey-streams-demo/`).
   * Each code example should have its own `requirements.txt` if new Python packages are introduced, and a small `README.md` explaining how to run it.
   * If you're adding a new concept (`.md` file), please try to follow the existing numbering scheme and content style.

5. **Test Your Changes:**
   
   * If you've modified code, make sure it runs correctly by starting the Valkey server with `docker compose up -d` and executing your script.
   * If you've modified Markdown, ensure it renders correctly on GitHub and all links are valid.

6. **Commit Your Changes:**
   Write clear, concise commit messages. A good commit message explains *what* change was made and *why*.
   
   ```bash
   git add .
   git commit -m "feat: Add new example for Valkey Streams"
   # or
   git commit -m "docs: Fix typo in 01-introduction-to-valkey.md"
   ```
   
   (Consider using [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for consistent messaging, but simple, descriptive messages are also fine).

7. **Push to Your Fork:**
   
   ```bash
   git push origin feature/my-new-example
   ```

8. **Open a Pull Request (PR):**
   
   * Go to your forked repository on GitHub.
   * You should see a "Compare & pull request" button or link. Click it.
   * Provide a clear title and description for your PR. Explain the problem it solves or the feature it adds. Link to any relevant issues.
   * Submit your PR.

## 🐛 Reporting Issues

If you encounter any bugs, errors, or have suggestions for improvements that you're not ready to implement yourself, please open an issue on GitHub.

* **Use a clear and descriptive title.**
* **Provide detailed steps to reproduce** if it's a bug.
* **Explain the expected behavior** and what actually happened.
* **Include any relevant error messages or screenshots.**

## ✍️ Style Guides and Conventions

To maintain consistency and readability:

* **Markdown:**
  * Use standard Markdown syntax.
  * Ensure proper code blocks with language highlighting (e.g., `` ```python ``).
  * Maintain the existing numbering and naming conventions for `concepts/` files (e.g., `06-new-topic.md`).
  * Reference images from the `assets/` directory using relative paths.
* **Python Code:**
  * Adhere to [PEP 8](https://peps.python.org/pep-0008/) style guidelines.
  * Include comments where necessary to explain complex logic.
  * Ensure code is runnable and self-contained within its example folder.

## 📜 License

By contributing your code or content to this repository, you agree to license your contributions under the [MIT License](LICENSE) that governs this project.

---

Thank you for helping to build a great Valkey learning resource!
