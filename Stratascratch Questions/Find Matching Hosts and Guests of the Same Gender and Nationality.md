[TOC]

# Find Matching Hosts and Guests of the Same Gender and Nationality



>  Original Prompt Title: Find matching hosts and guests in a way that they are both of the same gender and nationality

Find matching hosts and guests pairs in a way that they are both of the same gender and nationality. Output the host id and the guest id of matched pair.

`airbnb_hosts` table:

![image-20230313133534183](/Users/darren/Library/Application Support/typora-user-images/image-20230313133534183.png)

`airbnb_guests` table:

![image-20230313134711131](/Users/darren/Library/Application Support/typora-user-images/image-20230313134711131.png)

---

### Solution Steps

To find matching pairs of genders **and** nationality, join `airbnb_hosts` and `airbnb_guests` (final solution):

```mysql
select distinct(a.host_id), b.guest_id
from airbnb_hosts a
join airbnb_guests b on a.nationality = b.nationality
and a.gender = b.gender
```

There are repeat host IDs in `airbnb_hosts` (may represent hosts with multiple properties), so `distinct` is used to only match one pair.

---

### Alternative Solution

```mysql
select a.host_id, b.guest_id
from airbnb_hosts a, airbnb_guests b
where a.nationality = b.nationality and a.gender = b.gender
group by a.host_id, b.guest_id
```

This was my initial solution and is accepted, however the final solution is more optimized as the alternative is using an older implicit join syntax.  `host_id` serves as the primary group key (`guest_id` is grouped as well) which returns unique pairs.

---

### Notes

I had forgotten that varchar columns can be matched using join, so the alternative solution was a work around.

---

### Prompt Details

Prompt can be found on [Stratascratch](https://platform.stratascratch.com/coding/10078-find-matching-hosts-and-guests-in-a-way-that-they-are-both-of-the-same-gender-and-nationality?code_type=3), is medium rated, and found in Airbnb interviews.

Stratascratch does not offer an easier way to copy/paste table information, visit link to see full prompt details.