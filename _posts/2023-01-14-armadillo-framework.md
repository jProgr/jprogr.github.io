---
title: "Armadillo framework for software development"
permalink: /armadillo-framework
custom_date: "230114"
updated_date: "230124"
---

# Armadillo framework for software development

Here is a crazy idea. A simple framework for software development that focuses on removing bullshit. The work is organized in cards (or tickets) in a kanban board. There are four columns:

- Out.
- In.
- Dev.
- Done.

Non-developers create cards with stuff to be done and place them in either column Out or In. They can move them freely between them. Developers are free to take any card from those columns to work on using the following criteria, in order:

1. If there are cards in the column In, take any card from there.
2. If column In is empty, take any card from column Out.

That's it. Whether a card is at the top or bottom of a column is irrelevant. The only priority dimension is set by the columns Out and In.

If a developer is working on a card, this must be in the column Dev. This column is for any development work: coding, PRs, research, design, documentation, etcétera. Once any development work is finished, the card is moved to the column Done.

While taking cards from either Out or In, a card can be split into more cards. They will be placed where the source card was. This is done by a developer usually on cards that describe some task big and complex. Again, non-developers are free to move any of these cards between the first two columns.

Cards have the following fields:

- A unique identifier to aid communication.
- A short title. Max 100 chars.
- A description.
- A person responsible.
- File attachments.
- Comments.

A version history of changes is also useful but not obligatory.

The team is free to communicate as much as needed using any chat software. If a meeting is needed, every person gets the right to call for one each month. One a month, that's it. The meetings also have the following restrictions:

- Max 4 attendants.
- Max 30 minutes in length.

Optionally, people that do not use their right to call a meeting can get a bonus at the end of the month.

The things stated here are not the minimum, they are the maximum. There is not, and it is forbidden to add any of these:

- Periodic meetings. If you need one, use your once a month call.
- Story points or anything related.
- No estimation. If you need something done, put it in the column In and move other things to the Out column.
- There is no urgent issues, priority 0 or anything like that. We work with software. We are not firefighters, doctors nor lifesavers.
- An infinity of fields in cards. Cards do not have priority, labels, tags, components, weird card hierarchies, goals, types, epics, time tracking of any kind, dates, sprints nor any other bullshit. If you need to communicate something, use the fields already in the card.

The Armadillo framework is to be applied per discipline. For example, devs have their own board, QA can have their own board or use other framework or method to get things done. If something in the framework is not working for the team, feel free to remove it. You can remove whatever stated here. But you cannot add.

## FAQ

"Is this a joke?": 50/50.

"I need more fields in the cards, also I need another column for QA": No, you don't.

"People are not working on the things I need to": You are using the columns Out and In wrong. If you throw everything into the column In then you reduce the likelihood that a developer will work on the card you want. Remember, the order of cards in a column is irrelevant. Developers can grab any card from those first columns following the criteria. The only priority is set by column presence of Out and In. If there is only a card in the column In, then that one will be worked on next.

"Common, at least give me a daily status meeting": If you need to know what people are working on take a look at the column Dev.

"I don't like the names for the columns": Feel free to rename anything.

"Where do I put my backlog?": You can use the column Out for that.
