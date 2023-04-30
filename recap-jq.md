### JQ

See [Json and jq](https://programminghistorian.org/en/lessons/json-and-jq)

#### Core jq filters

```

cat jq_rkm.json | jq '.count'

cat jq_rkm.json | jq '..artObjects'

cat jq_rkm.json | jq -c '.artObjects[]'

cat jq_rkm.json | jq  '.artObjects[0]'

cat jq_rkm.json | jq  '.artObjects[] | .id'

cat jq_rkm.json | jq  '.artObjects[0] | .id'

# Let’s filter the Rijksmuseum JSON to only return the ids of objects that have at least one value assigned to their productionPlaces:
cat jq_rkm.json | jq  '.artObjects[] | select(.productionPlaces | length>=1 ) | .id '

cat jq_rkm.json | jq  '.artObjects[] | select(.principalOrFirstMaker | test("van") ) | {id: .id, artist: .principalOrFirstMaker} '

cat jq_rkm.json | jq  '.artObjects[] | {id: .id, title: .title}'

cat jq_rkm.json | jq  '.artObjects[] | [.id,.title]'

cat jq_rkm.json | jq  '.artObjects[] | [ .id, .title, .principalOrFirstMaker, .webImage.url ]'

cat jq_rkm.json | jq  -r '.artObjects[] | [ .id, .title, .principalOrFirstMaker, .webImage.url ] | @csv'

```

#### One-to-many relationships: Tweet hashtags

```
cat jq_twitter.json | jq  '{id: .id, hashtags: .entities.hashtags}'

cat jq_twitter.json | jq -c  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: .hashtags[].text}'

cat jq_twitter.json | jq -c  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: [.hashtags[].text]}'

cat jq_twitter.json | jq -c  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: [.hashtags[].text]} | {id: .id, hashtags: .hashtags | join(";")}'

cat jq_twitter.json | jq -c  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: [.hashtags[].text]} | {id: .id, hashtags: .hashtags | join(";")} | {id: .id, hashtags: .hashtags}'

cat jq_twitter.json | jq -r  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: [.hashtags[].text]} | {id: .id, hashtags: .hashtags | join(";")} | [ .id, .hashtags] | @csv'

cat jq_twitter.json | jq -c  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: .hashtags[].text} | [.id, .hashtags]'

cat jq_twitter.json | jq -r  '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: .hashtags[].text} | [.id, .hashtags] | @csv'

```

#### Grouping and Counting

`Slurp` tells jq to read every line of the input JSON lines and treat the entire group as one huge array of objects.

```

cat jq_twitter.json | jq -s   '.[0].user'

cat jq_twitter.json | jq -s   'group_by(.user)'

cat jq_twitter.json | jq -s   'group_by(.user) | .[] | {user_id: .[0].user.id, user_name: .[0].user.screen_name, user_followers: .[0].user.followers_count, tweet_ids: [.[].id | tostring] | join(";")}

cat jq_twitter.json | jq -s -r  'group_by(.user) | .[] | {user_id: .[0].user.id, user_name: .[0].user.screen_name, user_followers: .[0].user.followers_count, tweet_ids: [.[].id | tostring] | join(";")} | [.user_id, .user_name, .user_followers, .tweet_ids] | @csv'

cat jq_twitter.json | jq -s   '.[] | {id: .id, hashtag: .entities.hashtags} | {id: .id, hashtag: .hashtag[].text}

cat jq_twitter.json | jq -s   '[.[] | {id: .id, hashtag: .entities.hashtags} | {id: .id, hashtag: .hashtag[].text}] | group_by(.hashtag)'

cat jq_twitter.json | jq -s -r  '[.[] | {id: .id, hashtag: .entities.hashtags} | {id: .id, hashtag: .hashtag[].text}] | group_by(.hashtag) | .[] | {tag: .[0].hashtag, count: . | length} | [.tag, .count] | @csv'```
