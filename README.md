![Onify Blueprints](https://files.readme.io/8ba3f14-onify-blueprints-logo.png)

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)

# Onify Blueprint: Helix Showcase - Create ticket

This Blueprint provides a example for creating a ticket using a form page that integrates with the Onify Hub to run a workflow. Alongside this, you'll find demonstrations on validating form fields and uploading files to the Onify Hub. This serves as a guide to harnessing the power of our new framework, **Onify Helix**. Enjoy and stay tuned for more!

![Onify Blueprint: Onify Helix Showcase - Create ticket](blueprint.jpg "Blueprint")

## Requirements

* [Onify Hub](https://github.com/onify/install)
* [Onify Helix App](https://github.com/onify/helix-app-boilerplate)
* Onify Helix Components ^0.20.14

## Setup

### Setup Workflow

1. Copy `./hub/resources/*` to Onify Hub resources folder `./resources/`  
2. Sync resources with Git if needed
3. Create a new Workflow with key `create-ticket` and set `bpmnresource` to `create-ticket.bpmn`

### Setup Create ticket form

1. Copy `./helix/*` to your Onify Helix App root folder
2. Add to `create-ticket` page to `pages.yml` (see example below)

#### pages.yml

```yml
- name: create-ticket
  title: 'Create Ticket'
  path: /create-ticket
  import: ../src/custom/pages/create-ticket.vue
```

## Support

* Community/forum: https://support.onify.co/discuss
* Documentation: https://support.onify.co/docs
* Support and SLA: https://support.onify.co/docs/get-support

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.