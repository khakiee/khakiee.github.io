# Svelte blog template


This is a simple and minimalist blog template built using Svelte, This template provides an easy-to-use starting point for creating your own personal or professional blog.

## Features

- Static Deployment via github pages
- Responsive design for mobile and desktop devices
- Easy customization and theming
- Markdown support for blog posts
  - code highlight is not supported for now

## Getting Started

### Installation

1. Fork the repository:

```
git clone https://github.com/your-username/svelte-blog-template.git
```

2. Change env

you should change `.env` file to set your informations

```
VITE_USER_NAME=<your name>
VITE_GITHUB_URL=<your github url>
VITE_PROFILE_URL=<your profile avatar url>
VITE_EMAIL=<your email>
```

3. run dev

`npm run dev`
`yarn run dev`

Now, you can open your browser and navigate to `http://localhost:3000` to see your blog up and running.

## Customization

### Adding a New Blog Post

`yarn run new`
`npm run new`

This command create new post to `articles/<title>`. all spaces of title will be replaced with `-` and will be used to filename.
And also symbolic linked directory will created in `src/routes/posts/<title>`

### Deleting a Post

`yarn run delete`
`npm run delete`

### Theming and Styling

You can easily customize the look and feel of your blog by modifying the styles in `src/app.css` directory.

## !Deployment

You should activate `.github/workflows/.deploy.yml` via remove `.` of deploy script.

`mv .github/workflows/.deploy.yml .github/workflows/deploy.yml`

And you can also customize your build.

## TODO
- [ ] Add highlight to code block
- [ ] add home page
- [ ] add about page
- [ ] add ci pipeline

Happy blogging!
