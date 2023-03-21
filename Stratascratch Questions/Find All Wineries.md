[TOC]

# Find All Wineries Which Produces Wines by Possessing Aromas of Plum, Cherry, Rose, or Hazelnut

Find all wineries which produce wines by possessing aromas of plum,  cherry, rose, or hazelnut. To make it more simple, look only for singular form of the mentioned aromas.

*Example Description:* Hot, tannic and simple, with **cherry** jam and currant flavors accompanied by high, tart acidity and chile-pepper alcohol heat.

`winemag_p1` table:

![image-20230313022335917](/Users/darren/Library/Application Support/typora-user-images/image-20230313022335917.png)

---

### Solution Steps

Filter `description` by using `regexp`:

```mysql
select distinct(winery)
from winemag_p1
where description regexp "(plum|cherry|rose|hazelnut)"
```

The query above will output all instances where plum, cherry, rose, **or** hazelnut is referenced - however the prompt asks to look for singular form of the mentioned aromas and `regexp "(plum|cherry|rose|hazelnut)"` will return substring matches.  Words like **plum**s, **rose**s, P**rose**cco, **plum**my, etc will be picked up by the filter.

Setting word boundaries (`\\b`) will return whole words inside the alteration (final solution):

```mysql
select distinct(winery)
from winemag_p1
where description regexp "\\b(plum|cherry|rose|hazelnut)\\b"
```

Substring matches will be ignored and `regexp` is case sensitive.

---

### Alternative Solution

```mysql
select distinct(winery)
from winemag_p1
where description regexp "(plum|cherry|rose|hazelnut)([^a-z])"
```

This query filters `description` and ignores letters a-z (along with whitespace) before and after the list of words.

---

### Notes

My initial attempts were frustrating as I assumed `regexp` would return whole words:

```mysql
select distinct(winery)
from winemag_p1
where description regexp "plum|cherry|rose|hazelnut"
```

Stratascratch shows which rows are incorrect in the output, so I took a deeper look (see bold):

|      Winery      | Description                                                  |
| :--------------: | :----------------------------------------------------------- |
| Carpene Malvolti | This is a fresh and light Extra Dry style P**rose**cco with a luminous appearance and tonic bubbles. The aromas recall citrus, white flower and peach and the bubbly wine would taste great as an aperitivo. |
|   La Fiammenga   | Typical of this hot vintage, La Fiammenga is already quite mature. The  fruit is on the **plum**my side, it has the added complexity of licorice and road tar. On the palate, it is soft and rich but it does need a touch of acid to liven it up. Imported by Sapori Italiani Inc. |
| Finca El Origen  | This warm-weather Viognier is fleshy and chunky on the bouquet, with generic white-fruit aromas that suggest gardenias. A **plum**p palate is low on acid and heavy in weight; flavors of melon and green herbs finish soft and could turn flabby in a short time. |

If word boundaries aren't set (or `[^a-z]`), substring matches will be outputted and the prompt does ask to only return singular forms of the mentioned aromas (capitalized instances and substring matches should be ignored).

---

### Prompt Details

Prompt can be found on [Stratascratch](https://platform.stratascratch.com/coding/10026-find-all-wineries-which-produce-wines-by-possessing-aromas-of-plum-cherry-rose-or-hazelnut?code_type=3), is medium rated, and found in Wine Magazine interviews.

Stratascratch does not offer an easier way to copy/paste table information, visit link to see full prompt details.