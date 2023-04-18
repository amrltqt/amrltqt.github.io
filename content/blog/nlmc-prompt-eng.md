+++
title = "NLMC - a simple practical use case"
date = "2023-03-08"
+++

In this post, we'll explore how to use Large Language Models (LLMs) to build a Natural Language, Machine Control (NLMC) process, enhancing user experience in software applications. LLMs enable software to better understand human language, and when combined with effective summarization, they can help create natural language interfaces for today's APIs.

For instance, consider building a search engine for products. Traditional algorithms parse user queries, extract keywords, and generate a query for the product search engine. However, language is about conveying meaning, not just keywords. LLMs can handle more natural queries, such as "I want to sleep better," and generate relevant product suggestions.

Let's have a try.

```prompt
Imagine that you're helping people to buy products on a marketplace.
Propose products categories as a list for the request "I want to sleep better"
```

```answer
Sure, here are some product categories that could help someone sleep better:

Bedding: High-quality sheets, pillows, and comforters can make a huge difference in the quality of your sleep.

Sleep aids: There are a variety of products designed to help you fall asleep and stay asleep, such as white noise machines, essential oils, sleep masks, and earplugs.

Mattresses and mattress toppers: A good mattress or mattress topper can provide the support and comfort you need for a good night's rest.
```

It's definitely really interesting to rely on the LLM to identify what to look for (bonus, you can provide the _WHY_ to your customers).

In order to easily postprocess the information you can be a bit more assertive on the way you format the answer

Adding the following instruction in the end of the the prompt will help to secure the output format

```prompt
...
format the answer in JSON with category as a key and the justification as value
```

```json
{
  "Pillows": "A good pillow can significantly improve the quality of your sleep by providing proper support and alignment for your head, neck, and shoulders. There are pillows available in the market that cater to specific needs, such as memory foam, cooling gel, and orthopedic pillows.",
  "Mattresses": "A comfortable and supportive mattress is essential for a good night's sleep. A mattress that is too hard or too soft can cause discomfort and disrupt sleep. Look for a mattress that suits your sleeping position and preferences, such as memory foam, latex, or hybrid mattresses",
  "Bedding": "Choosing the right bedding is important for a comfortable sleep. This includes sheets, blankets, comforters, and duvets. Look for materials that are breathable, such as cotton or linen, to keep you cool during the night."
}
```

You now have your categories in a format that is easy to manipulate with any modern programming language. You can reuse the keys to look for products in your search engine and complete the user request with products and a feedback from the LLM.

It improve a lot the user experience while reusing your existing API code. Those are exactly the objectives of the NLMC approach.
