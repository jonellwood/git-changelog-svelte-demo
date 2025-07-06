# SvelteKit Changelog Renderer

A modern, responsive changelog renderer built with SvelteKit. This template demonstrates how to display changelog data using the `git-changelog-manager` package with a beautiful, Apple-inspired design.

## Demo

🌐 **[Live Demo](https://svelte.jonellwood.dev){:target="_blank"}** - See it in action!

[![Netlify Status](https://api.netlify.com/api/v1/badges/49bbde6d-4b1f-4967-b5d9-2c01bf5d0d66/deploy-status)](https://app.netlify.com/projects/svelte-changelog-demo/deploys)

## Features

- 📱 **Responsive Design** - Works on desktop, tablet, and mobile
- 🌙 **Dark Mode** - Modern dark theme with accent colors
- 🔍 **Search & Filter** - Find releases by version, summary, or content
- 📊 **Statistics Dashboard** - Overview cards showing release metrics
- 📋 **Multiple Views** - Switch between grid and list layouts
- 🎨 **Beautiful UI** - Phosphor Icons and modern styling
- 📝 **Release Details** - Modal view with formatted markdown content
- 🗂️ **Sidebar Navigation** - Organized by month with quick stats

## Getting Started

### Prerequisites

- Node.js 18+ 
- npm or yarn

### Installation

1. Clone the repository:
```bash
git clone https://github.com/jonellwood/git-changelog-renderer-examples.git
cd git-changelog-renderer-examples/sveltekit-changelog
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npm run dev
```

4. Open your browser to `http://localhost:5173`

## Usage

### With git-changelog-manager

This template is designed to work with the `git-changelog-manager` package. Simply replace the sample data in `src/lib/data/sampleData.js` with data from your changelog manager:

```javascript
import { changelogData } from 'git-changelog-manager';

// Use the data in your Svelte component
```

### Customizing Data

The changelog expects data in this format:

```javascript
{
  releases: [
    {
      version: '1.2.0',
      date: '2025-01-04',
      summary: 'Major feature release',
      tag: 'feature', // feature, patch, hotfix, major
      changeCount: 12,
      rawContent: '- Feature 1\n- Feature 2\n...'
    }
  ],
  stats: {
    totalReleases: 6,
    totalChanges: 67,
    latestVersion: '1.2.0'
  }
}
```

## Components

- **ChangelogManager** - Main component handling search, filtering, and display
- **Header** - Page title and description
- **Sidebar** - Navigation and statistics
- **Sample Data** - Mock changelog data for demonstration

## Styling

The project uses modern CSS with:
- CSS custom properties for theming
- `color-mix()` for dynamic color variations
- Flexbox and Grid layouts
- Smooth transitions and hover effects
- Mobile-first responsive design

## Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run check` - Run Svelte checks

## License

MIT License - feel free to use this template for your projects.
