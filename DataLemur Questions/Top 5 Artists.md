[TOC]

# Top 5 Artists

Assume there are three Spotify tables containing information about the artists, songs, and music charts. Write a query to determine the top 5 artists whose songs appear in the Top 10 of the `global_song_rank` table the highest number of times. 

Output the top 5 artist names in ascending order along with their song appearances ranking (not the number of song appearances, but the rank of who has the most appearances). The order of the rank should take precedence.  Example inputs of all tables will be at the bottom of the page.

Assumptions:

* If two artists' songs have the same number of appearances, the artists should have the same rank.
* The rank number should be continuous (1, 2, 2, 3, 4, 5) and not skipped (1, 2, 2, 4, 5).

`artists` table:

| Column Name |  Type   |
| :---------: | :-----: |
|  artist_id  | integer |
| artist_name | varchar |

`songs` table:

| Column Name |  TypE   |
| :---------: | :-----: |
|   song_id   | integer |
|  artist_id  | integer |

`global_song_rank` table:

| Column Name |         Type          |
| :---------: | :-------------------: |
|     day     |    integer (1-52)     |
|   song_id   |        integer        |
|    rank     | integer (1-1,000,000) |

---

### Solution Steps

Finding artists that has appeared in the top 10:

```postgresql
select c.artist_name, a.rank
from global_song_rank a
join songs b on a.song_id = b.song_id
join artists c on b.artist_id = c.artist_id
where rank between 1 and 10
```

Remember that the prompt asks to determine which artists has appeared in the **top 10**, so the query above filters `rank` between 1-10 and lists the artists by name.  This would be used as a CTE to rank artists by the number of appearances from this table.

`top_10` holds the query that finds artists that has appeared in the top 10 and is ranked by number of appearances:

```postgresql
with top_10 as (
	select c.artist_name, a.rank
	from global_song_rank a
	join songs b on a.song_id = b.song_id
	join artists c on b.artist_id = c.artist_id
	where rank between 1 and 10
)

select artist_name, dense_rank() over (order by count(*) desc) as ranking
from top_10
group by artist_name
```

Artists' rank number should be **continuous**, so the dense_rank() function is used to rank artists by the number of appearances in `top_10` in descending order.

The main query ranks **all** artists with the number of appearances in the top 10 (descending order) and the prompt requests to output only the top 5 ranking artists.  The query can be used as a subquery  (`s1`) and the where clause is used to filter by `ranking` (final solution):

```postgresql
with top_10 as (
	select c.artist_name, a.rank
	from global_song_rank a
	join songs b on a.song_id = b.song_id
	join artists c on b.artist_id = c.artist_id
	where rank between 1 and 10
)

select artist_name, ranking
from (
	select artist_name, dense_rank() over (order by count(*) desc) as ranking
  from top_10
  group by artist_name
) as s1
where ranking <= 5
```

The limit clause cannot be used here in the test case that two (or more) artists share the same number of appearances.  If `limit 5` were used here, the output of ranks will look like:

| artist_name | ranking |
| :---------: | :-----: |
|  Bad Bunny  |    1    |
| Ed Sheeran  |    2    |
|    Adele    |    3    |
|  Lady Gaga  |    3    |
| Katy Perry  |    4    |

This submission would be incorrect as only ranks 1-4 is outputted.

---

### Alternative Solution

```postgresql
with top_10 as (
	select c.artist_name, dense_rank() over (order by count(a.*) desc) as ranking
  from global_song_rank a
  join songs b on a.song_id = b.song_id
  join artists c on b.artist_id = c.artist_id
  where rank between 1 and 10
  group by c.artist_name
)

select * from top_10
where ranking <= 5
```

Found a solution by Anand Siva (comment section) and it should be more optimized since a subquery isn't used.  Siva's syntax has been changed to fit my preferences.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/top-fans-rank), is medium rated, and found in Spotify interviews.

---

### Example Inputs

`artists` example input:

| artist_id | artist_name |
| :-------: | :---------: |
|    101    | Ed Sheeran  |
|    120    |    Drake    |

`songs` example input:

| song_id | artist_id |
| :-----: | :-------: |
|  45202  |    101    |
|  19960  |    120    |

`global_song_rank` example input:

| day  | song_id | rank |
| :--: | :-----: | :--: |
|  1   |  45202  |  5   |
|  3   |  45202  |  2   |
|  1   |  19960  |  3   |
|  9   |  19960  |  15  |

