# Falcode Documentation Page

This project is based on just-the-docs template [link](https://github.com/just-the-docs/just-the-docs-template)

Browse "just the docs" [documentation](https://just-the-docs.github.io/just-the-docs/) to learn more about how to use this theme.

## Prerequisites

Before running this project, you need to have the following installed:

- **Ruby** (version 2.7.0 or higher)
- **Jekyll** (will be installed via bundler)

### Installing Ruby and Jekyll

#### On Ubuntu/Debian:
```bash
sudo apt update
sudo apt install -y ruby ruby-dev bundler jekyll
```

#### On macOS:
```bash
# Using Homebrew
brew install ruby

# Or using rbenv
rbenv install 2.7.0
rbenv global 2.7.0
```

#### On Windows:
Download and install Ruby from [rubyinstaller.org](https://rubyinstaller.org/)

## Getting Started

1. **Install dependencies:**
   ```bash
   bundle install
   ```

2. **Run the development server:**
   ```bash
   bundle exec jekyll serve --livereload
   ```

3. **Open your browser:**
   Navigate to `http://localhost:4000` to view the documentation site.

## Development

- The site will automatically reload when you make changes to files
- Jekyll will watch for file changes and rebuild the site
- To stop the server, press `Ctrl+C` in the terminal

## Building for Production

To build the site for production:

```bash
bundle exec jekyll build
```

This will generate the static site in the `_site` directory.
