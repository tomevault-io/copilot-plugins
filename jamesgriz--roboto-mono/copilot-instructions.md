## roboto-mono

> When creating sanity schema make sure to include an appropriate icon for the schema using lucide-react or sanity icons as a fallback. Make sure you're always using the Sanity typescript definitions if it's a ts file. Here's an example, make sure you use a defineField on every field and a defineType throughout the whole type. If you don't see any arrayMembers, don't import it:

----------------------------------SANITY RULES----------------------------------
When creating sanity schema make sure to include an appropriate icon for the schema using lucide-react or sanity icons as a fallback. Make sure you're always using the Sanity typescript definitions if it's a ts file. Here's an example, make sure you use a defineField on every field and a defineType throughout the whole type. If you don't see any arrayMembers, don't import it:

import {defineField, defineType, defineArrayMember} from 'sanity' if it's a .ts file. 

 defineType({
   type: 'object',
   name: 'custom-object',
   fields: [
     defineField({
       type: 'array',
       name: 'arrayField',
       title: 'Things',
       of: [
         defineArrayMember({
           type: 'object',
           name: 'type-name-in-array',
           fields: [defineField({type: 'string', name: 'title', title: 'Title'})],
         }),
       ],
     }),
   ],
 })

When writing any Sanity schema that matches with any part of the keywords below, use these as a template including the description. Make sure description is always above type. Whenever generating sanity schema ensure you're always including a description, as well as the name, title and type, have a best guess as to what the field does and describe that functionality in the simplest way possible to a non-technical user.

{
	name: 'eyebrow',
	title: 'Eyebrow',
	description: 'The smaller text that sits above the title to provide context',
	type: 'string',
},
{
	name: 'title',
	title: 'Title',
	description: 'The large text that is the primary focus of the block',
	type: 'string',
},
{
	name: 'isHeadingOne',
	title: 'Is it a <h1>?',
	type: 'boolean',
	description:
	'By default the title is a <h2> tag. If you use this as the top block on the page, you can toggle this on to make it a <h1> instead',
	initialValue: false,
},
{
  name: 'richText',
  title: 'Rich Text',
  description: 'Large body of text that has links, ordered/unordered lists and headings.',
	type: 'richText',
},
{
	name: 'buttons',
	title: 'Buttons',
	description: 'Add buttons here, the website will handle the styling',
	type: 'array',
	of: [{type: 'button'}],
},
{
    name: 'image',
    title: 'Image',
    type: 'image',
    fields: [
      {
        name: 'alt',
        type: 'string',
        description:
          "Remember to use alt text for people to be able to read what is happening in the image if they are using a screen reader, it's also important for SEO",
        title: 'Alt Text',
      },
    ],
  },


Always use tailwind to style if the codebase is using it. Always use functional react. 
Whenever we use a central container it will always have a default margin-y of 16 and appropriate padding, it generally is max-w-7xl unless it's specified otherwise and mx-auto to central it. Here's an example:
<section className="my-16 px-4">
	<div className="mx-auto max-w-7xl>
	…
	</div>
</section>
If something has a background we typically display it like this and break pattern when it's a page builder block. Take the example above and slightly tweak it to this
<section className="my-8 px-4">
	<div className=“py-8”> //Background layer
		<div className="mx-auto max-w-7xl>
		…
		</div>
	</div>
</section>

We always prefer to use grid instead of flex unless it’s two sibling tags, in which case only then would you use a flex Here's an example where we would use flex:
<div>
	<img/>
	<p>Some text</p>
<div>
We always use appropriate semantic html

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JamesGriz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
