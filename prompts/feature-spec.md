<summary>
It shall be possible to assign one example image to a prompt. Then the example image can be used to show the outcome of a image manipulation in the `ImageWizardDialog` and potential in other components too.

First lets implement the logic to be able to assign and store an example image for a prompt.
</summary>

<requirements>
- Extend the prompt model to handle an example image
- The example image shall be stored directly in the database as blob
- Add a new column to the prompt table to store the example image
- Extend the admin page to show and edit the example image
</requirements>

<files>
## Probably involved files
- Admin Dashboard: @src/...
- Prompt Model: @src/...
</files>

