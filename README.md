# TeXstudio_CurrencyCodes

A macro for TeXstudio that allows the user to look up a country, currency, or ISO 4217 code, and will tell them what all corresponding information (current as of Jan 1, 2023) for that country's currency is, giving them the option to insert the code where the cursor is.

The search functionality works by first checking against selected text, or, if it can't find anything, by allowing the user to input the name of a country, currency, or an ISO code before checking. While most countries have complex names in the ISO database, the macro works with wildcards to ensure that common names (or even just snippets thereof) can be used in the search. Various common alternate English names for countries, as well as abbreviations, are also listed, though this may be incomplete.
