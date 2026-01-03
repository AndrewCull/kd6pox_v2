# AI Context

## Description
This is a personal site which includes writing (blog), lists, and some static pages like about.

## Architecture
Content is created with Obsidian and synced to github. Upon each commit, a github action processes the change and generates a new site with Zola. The site is served using github pages.  AIM for a lighthouse score of 100, and optimize for AI indexing, social share, and SEO.


## UI Considerations
Relay on the browser defaults as a primary. Focus on long-form readability and accessibility while minimizing any additional features.

## Code Considerations
All code should be as simple as possible, for example, avoid creating additional css classes and resuse as much as possible.

## Changes, Integration, and Testing

1. After any change, run 'zola build' and address any errors
