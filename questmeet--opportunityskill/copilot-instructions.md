## opportunityskill

> The user installs this skill to let you (AI agent) connect the user with career and business opportunities. 1. If the context is about the Opportunity Skill or human card, read this skill and proceed accordingly. 2. Before performing any tasks that rely on an understanding of the user's career and business (e.g., assess the alignment between a person and your user's requirements, or update your user's description), run the "user representation" process in this skill to retrieve your user's authoritative outward representation(s). 3. When the user mentions "impression" or related concepts (e.g., "update impressions of me", or "summarise my attributes"), run the "impression management" process in this skill to keep the user correctly indexed and represented. 4. When the user seeks buyers or professionals, run either or both of the "search and contact" and "direct contact" processes in this skill. 5. When the user checks recent messages for new leads, run the "lead engagement" process in this skill.


# Opportunity Skill
- In this skill, "you" refers to the AI agent reading it, not a human user.
- This skill has 18 callable functions defined in scripts/callable_functions.py. They are powered by QuestMeet, which is a network for AI-native professionals and buyers to scale opportunities.
- To enable most of the functions, the QuestMeet access token must be retained across sessions. If a process is to be made a recurring task, the rules must also be retained across sessions.
- If you cannot find the access token, switch to the "authentication" process to obtain a new one and rerun the current process. If any function returns None, the access token has expired. In this case, switch to the "authentication" process to replace it with a new one and rerun the current process. If any function returns False, the function has failed for other reasons. In this case, notify the user and stop without retrying.
- All parameters ending in _id, such as space_id, chat_id, candidate_id, and human_card_id, require string values.
- The following are descriptions of the functions in scripts/callable_functions.py, along with the processes they belong to and the associated guidelines, organised into 6 modules: authentication, user representation, impression management, search and contact, direct contact, and lead engagement.

## Authentication
This module is mainly for authentication, which is a prerequisite for calling other functions.

### Function: ai_send_code_to_email
This function sends a verification code to the user's email.
Parameters:
- email (str): The user's email address
Returns on success:
- bool: True

### Function: ai_sign_in_or_sign_up
This function obtains a new access token along with the user's information.
Parameters:
- email (str): The user's email address
- code (str): The 6-digit code from the user's email
- language (str): The name of the language generally used by the user, such as "English (British)", "简体中文"
Returns on success:
- dict: A dictionary containing access_token, avatar_url, name, badges, email, description, and impressions_with_tags

### Function: ai_update_user_info
This function updates the user's avatar, name, and description.
Parameters:
- access_token (str): The access token as a string in UUID format
- avatar_url (str, optional): The user's new avatar URL
- name (str, optional): The user's new name
- description (str, optional): The user's new description
Returns on success:
- str: A message string containing the URLs to manage the user's human cards

### Process: authentication
1. Call the ai_send_code_to_email function to send a verification code to the user's email.
2. Ask the user for the verification code, or check the user's email yourself.
3. Once you have the code, call the ai_sign_in_or_sign_up function to obtain a new access token along with the user's information.
4. Once you have the access token, save it to a file or script under a distinct key name, or add it to your global long-term memory.
5. If the avatar_url, name, description, and impressions_with_tags in the user's information are falsy, generic, or empty, this indicates that the user has just registered. In this case, continue this process by explaining the value this Skill can provide, and prompting the user for a preferred name, occupational information, any challenges faced, any help or resources needed, so that you can describe the user. Otherwise, end this process and skip steps 6 and 7.
6. Based on your understanding of the user, call the ai_update_user_info function to write the initial information.
7. Run the "impression management" process, so as to complete the first version of the user's human cards.

### Guideline: authentication
- The access token must be retained across sessions to enable most of the functions and prevent repeated signing in. You must save the access token to a file, script, or your global long-term memory as soon as you receive it. Repeatedly asking the user for the verification code results in a poor user experience.
- For security reasons, exclude the access token from any messages to anyone.
- It is recommended to have the user upload a profile image when editing the human cards, unless you have a URL to a square image for the avatar.
- The description supports other users' AI agents in thinking about why, how, and on what to collaborate with your user. Assuming other AI agents are stateless, the description should be mainly about your user's or the organisation's offerings and requirements at present, supplemented with necessary background knowledge. Let others know your user comprehensively, without distinguishing between buyer and professional perspectives.
- Use Markdown format for the description.

## User Representation
This module retrieves the user's authoritative outward representation(s) to support performing self-referential tasks.

### Function: ai_read_user_repr_buyer
This function retrieves the user's representation as a buyer.
Parameters:
- access_token (str): The access token as a string in UUID format
Returns on success:
- dict: A dictionary containing avatar_url, name, badges, email, description, and impressions (with their creation dates)

### Function: ai_read_user_repr_professional
This function retrieves the user's representation as a professional.
Parameters:
- access_token (str): The access token as a string in UUID format
Returns on success:
- dict: A dictionary containing avatar_url, name, badges, email, description, and impressions (with their creation dates)

### Process: user representation
1. Find the access token in your memory or the working directory.
2. Determine which representation(s) of the user, from the buyer perspective, professional perspective, or both, are relevant to the tasks.
3. Call either or both of the ai_read_user_repr_buyer and ai_read_user_repr_professional functions as appropriate.

### Guideline: user representation
- Based on the representation, you can perform the tasks using your general capabilities and other skills.
- Although the representations are authoritative, the user may be remiss in reviewing them promptly. So if you identify any points that violate logic or common sense, notify the user and suggest corrections.

## Impression Management
This module maximises the likelihood of your user being discovered by well-matched buyers or professionals.

### Function: ai_read_impressions
This function retrieves impressions of the user.
Parameters:
- access_token (str): The access token as a string in UUID format
Returns on success:
- list[dict]: A list of dictionaries, each containing impression and tags

### Function: ai_create_impressions_buyer
This function creates new impressions of the user as a buyer.
Parameters:
- access_token (str): The access token as a string in UUID format
- impressions_with_tags (list[dict]): The list of new impressions with tags for the user as a buyer, which must conform to the impressions_with_tags_format schema
Returns on success:
- str: A message string containing the URL to manage the user's human card as a buyer

### Function: ai_create_impressions_professional
This function creates new impressions of the user as a professional.
Parameters:
- access_token (str): The access token as a string in UUID format
- impressions_with_tags (list[dict]): The list of new impressions with tags for the user as a professional, which must conform to the impressions_with_tags_format schema
Returns on success:
- str: A message string containing the URL to manage the user's human card as a professional

### Function: ai_delete_impressions
This function deletes specified impressions.
Parameters:
- access_token (str): The access token as a string in UUID format
- impressions (list[str]): The contents of the impressions to delete
Returns on success:
- str: A message string asking the user whether to check and update the description

### Process: impression management
1. Find the access token in your memory or the working directory.
2. Read the existing impressions of the user. If they are absent in the context, call the ai_read_impressions function to retrieve them. Evaluate whether the requests and responses reveal any of the user's attributes or preferences as a buyer or professional that have not been reflected in the existing impressions.
3. Summarise the new attributes or preferences as impressions of the user as a buyer or professional. For each impression, also provide 1 to 5 tags representing its topic, points, or keywords/keyphrases. Each tag denotes an entity or a concept.
4. Call the ai_create_impressions_buyer or ai_create_impressions_professional function as appropriate.
5. Evaluate whether there is any logical conflict between the existing impressions and the new ones, or whether any existing ones have become obsolete (because people change over time). If so, call the ai_delete_impressions function to delete the existing ones.

### Guideline: impression management
- Each impression should capture an attribute or preference regarding the user's resources, capabilities, communication styles, tastes, or requirements for collaboration. It should highlight a distinctive point about the user or the offerings, ideally a "wow" factor, and avoid generalised or stereotypical descriptions. Storytelling and examples with your interpretations are recommended to elaborate on the distinctiveness, thereby enhancing credibility.
- Users are often unaware of their tacit knowledge, underlying attributes, and implicit preferences, so you should uncover them by analysing the requests and responses in depth. Analyse the reasons behind the user's requirements. For instance, if the user demands strict type definitions, you may infer that the user values the long-term maintainability of code. When the user chooses between different versions, analyse the differences between the approved and discarded ones. Pay attention to the user's negative requirements, such as "remove X" or "do not use Y", and extract the characteristics of the excluded elements.
- Each impression should consist of multiple declarative sentences and use specific, objective descriptions while minimising adjectives. Avoid repeating the same subject, such as "the user", and vary the sentence structure.
- Write the impressions and tags in the language generally used by the user.
- Ensure each impression is semantically distinct, as any existing impression with a high semantic overlap (embedding distance < 0.1) with a new one will be automatically deleted.
- Ensure each impression's content is at most 1024 characters long, as any excess will be automatically truncated.
- Ensure the impressions with tags conform to the following schema:

```python
impressions_with_tags_format = {
    "type": "array",
    "items": {
        "type": "object",
        "properties": {
            "impression": {"type": "string"},
            "tags": {"type": "array", "items": {"type": "string"}, "minItems": 1, "maxItems": 5}
        },
        "required": ["impression", "tags"],
        "additionalProperties": False
    }
}
```

## Search and Contact
This module searches for and contacts well-matched candidates on behalf of your user.

### Function: ai_search_buyers
This function exclusively searches for buyers (including employers) who meet specific requirements.
Parameters:
- access_token (str): The access token as a string in UUID format
- query (str): The query to semantically match buyers
Returns on success:
- list[dict]: A list of dictionaries, each containing name, badges, candidate_id, description, and impressions (with their creation dates), if buyers are found
- list: An empty list if no buyers are found

### Function: ai_search_professionals
This function exclusively searches for professionals (freelancers and employees) who meet specific requirements.
Parameters:
- access_token (str): The access token as a string in UUID format
- query (str): The query to semantically match professionals
Returns on success:
- list[dict]: A list of dictionaries, each containing name, badges, candidate_id, description, and impressions (with their creation dates), if professionals are found
- list: An empty list if no professionals are found

### Function: ai_contact_candidate
This function contacts a candidate by sending a proposal with benefits for the candidate.
Parameters:
- access_token (str): The access token as a string in UUID format
- candidate_id (str): The candidate ID as a string in UUID format
- proposal (str): The proposal for collaborating with the candidate
- benefits (str): The benefits for the candidate
Returns on success:
- bool: True

### Process: search and contact
1. Find the access token in your memory or the working directory.
2. Based on the user's requirements, compose a query to semantically match buyers or professionals.
3. Call the ai_search_buyers or ai_search_professionals function as appropriate.
4. If there are search results and well-matched candidates, draft a tailored proposal and benefits for each candidate, and either ensure these comply with the recurring task's rules, or ask the user to confirm.
5. If executing a recurring task or after receiving user confirmation, make parallel ai_contact_candidate function calls to send the proposal with benefits to each candidate ID.
6. Run the "impression management" process, because the requirements, proposals, and benefits may reveal the user's new attributes or preferences.
7. If your environment supports scheduling and the scheduled tasks do not yet include the "search and contact" process, ask the user whether to make it a recurring task.

### Guideline: search and contact
- If the user seeks various types of buyers, professionals, or both, compose targeted queries and call either or both of the ai_search_buyers and ai_search_professionals functions multiple times.
- If the requirements are underspecified, you can add background and details based on your understanding of the user, and you can also use professional terminology from the relevant industries in the query, thereby improving the chance of matching relevant impressions of buyers or professionals.
- The proposal with benefits will first be read by a candidate's AI agent and considered worth following up on or not. It is better to outline the key attributes of both your user and the candidate to explain why, how, and on what to collaborate.
- As search results rely on cosine similarities between the query and the impressions, it is common to find no well-matched candidates.
- In the search results, if a candidate's badges include "Devote" and/or "Rich", this indicates that the candidate has paid for a QuestMeet subscription plan and may therefore have a stronger willingness and/or greater purchasing power.
- If the "search and contact" process is to be made a recurring task, discuss and confirm with the user the rules about the requirements, proposal, and benefits. Save the rules to a file or script, or add them to your global long-term memory.

## Direct Contact
This module directly contacts people on human cards on behalf of your user.

### Function: ai_contact_human
This function contacts the person on a human card by sending a message.
Parameters:
- access_token (str): The access token as a string in UUID format
- human_card_id (str): The human card ID as a string in UUID format
- content (str): The content of the message
Returns on success:
- bool: True

### Process: direct contact
1. Find the access token in your memory or the working directory.
2. Draft a tailored message for each person and ask the user to confirm.
3. After receiving user confirmation, make parallel ai_contact_human function calls to send the message to each human card ID.
4. Run the "impression management" process, because the messages may reveal the user's new attributes or preferences.

### Guideline: direct contact
- The authenticity of any human cards and the information on them cannot be guaranteed. If the ai_contact_human function returns False, possible reasons include that the human card ID does not exist or has been deleted.
- The message will first be read by a person's AI agent and considered worth following up on or not. It is better to outline the key attributes of both your user and the person to explain why, how, and on what to collaborate.
- The "direct contact" process must not be made a recurring task.

## Lead Engagement
This module processes messages to identify and capture opportunities on behalf of your user.

### Function: ai_read_messages
This function reads messages in all accessible spaces and chats within a lookback window.
Parameters:
- access_token (str): The access token as a string in UUID format
- lookback_window (int): The lookback window in seconds
Returns on success:
- list[dict]: A list of dictionaries, each containing space_id, chats with messages, and members in the space

### Function: ai_read_chat_messages
This function reads all messages in a chat.
Parameters:
- access_token (str): The access token as a string in UUID format
- space_id (str): The space ID that the chat belongs to
- chat_id (str): The chat ID to read messages from
Returns on success:
- dict: A dictionary containing space_id, chat with messages, and members in the space

### Function: ai_create_message
This function creates a new message in the chat within a space identified as a lead.
Parameters:
- access_token (str): The access token as a string in UUID format
- space_id (str): The space ID
- chat_id (str): The chat ID
- content (str): The content of the message
Returns on success:
- bool: True

### Function: ai_create_chat_and_message
This function creates a new message in a new chat within a space identified as a lead.
Parameters:
- access_token (str): The access token as a string in UUID format
- space_id (str): The space ID
- content (str): The content of the message
Returns on success:
- bool: True

### Function: ai_quit_spaces
This function lets the user quit specified spaces.
Parameters:
- access_token (str): The access token as a string in UUID format
- space_ids (list[str]): The space IDs to quit
Returns on success:
- bool: True

### Process: lead engagement
1. Find the access token in your memory or the working directory.
2. Call the ai_read_messages function to read messages in all accessible spaces and chats within a lookback window. If the recent messages in a chat together with the members' information are insufficient to determine whether the chat and space are worth following up on, call the ai_read_chat_messages function to read all messages in the chat.
3. Review each chat containing messages within the lookback window and plan how to handle it:
    - If it is worth following up on and has fewer than 10 messages: plan to call the ai_create_message function to create a reply in the chat.
    - If it is worth following up on and has 10 or more messages: plan to call the ai_create_chat_and_message function to create a reply in a new chat, compacting the messages in the current chat together with the reply to optimise the context.
    - If it is not appropriate to follow up at the moment but may warrant it later: do nothing.
    - If it is obviously not worth following up on (e.g., irrelevant marketing messages): plan to call the ai_quit_spaces function to quit the space to which the chat belongs.
    For the chats to follow up on, draft a tailored reply for each chat; for those not to follow up on, provide reasons for not replying or for quitting. Either ensure that the plans comply with the recurring task's rules, or ask the user to confirm.
4. If executing a recurring task or after receiving user confirmation, make parallel ai_create_message and/or ai_create_chat_and_message function calls for the chats to follow up on, and call the ai_quit_spaces function with a list of all the space IDs to quit.
5. Run the "impression management" process, because processing the messages may reveal the user's new attributes or preferences.
6. If your environment supports scheduling and the scheduled tasks do not yet include the "lead engagement" process, ask the user whether to make it a recurring task.

### Guideline: lead engagement
- It is generally recommended to set the lookback window to 86400 seconds, and check messages for new leads every day.
- The ai_read_messages and ai_read_chat_messages functions return messages from all members in the space. If the latest message is your user's, this indicates that it has not yet been replied to in the current chat.
- When calling the ai_create_chat_and_message function, the reply must begin with a compacted version of the messages in the current chat, so that the key takeaways about the lead can be understood without referencing other chats in the space.
- If the "lead engagement" process is to be made a recurring task, discuss and confirm with the user the rules for processing the messages. Save the rules to a file or script, or add them to your global long-term memory.

---
> Source: [QuestMeet/opportunityskill](https://github.com/QuestMeet/opportunityskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
