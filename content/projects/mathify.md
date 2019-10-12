---
title: "A simple LaTeX equation editor: Mathify"
description: "Create LaTeX equations like they are a piece of 🍰"

draft: false

categories:
- LaTeX
- Mathematics
- Mathify
tags:
- Development
- LaTeX
---

# Equations. Made beautifully simple.

During a recent university project our team had to write a lot of complex mathematical equations for
our project. Since our work is being written in the online LaTeX editor Overleaf, writing
complicated equations by hand is not really a pleasant experience.

I quickly realised how useful it would be to be able to write mathematical expressions on a computer
as easily as you would on a piece of paper, with an automatic conversion to LaTeX so people can
directly insert it into the project files.

So that is exactly what I did. I looked for a day or two on the web to find a solid and simple
implentation of an equation editor and this resulted in me using [MathQuill](http://mathquill.com/)
under the hood to allow users to write their math.

When I first showed it to the members in my project group, they were really happy because it
signifcantly sped up the process of writing. However, they also had some good remarks, such as the
addition of some additional explanation and the support for more Greek characters. I listened and
added all of this to the editor.

I decided to dub the page 'Mathify'. You can find the editor [here](https://douwe.xyz/math/).