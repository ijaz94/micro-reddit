# Micro-Reddit

A simple Reddit-like Rails application built to practice data modeling, validations, and associations between `User`, `Post`, and `Comment` models.

---

## 📋 Overview

This project was built following a step-by-step exercise to understand:
- Generating a Rails app and models
- Writing and running migrations
- Adding validations to models
- Testing validations in the Rails console
- Setting up associations (`has_many`, `belongs_to`)
- Building and saving records via associations

Users can:
- Be created (no login/logout or password security yet)
- Submit links ("posts")
- Comment on posts (comments belong to a post, not to other comments)

---

## 🛠️ Setup

### 1. Create the Rails App

```bash
rails new micro-reddit
cd micro-reddit
```

Uses the default SQLite3 database — no configuration changes needed.

---

## 🧑 User Model

### Generate the model

```bash
rails generate model User username:string
rails db:migrate
```

### Validations added in `app/models/user.rb`

```ruby
class User < ApplicationRecord
  has_many :posts
  has_many :comments

  validates :username, presence: true, uniqueness: true, length: { minimum: 3, maximum: 20 }
end
```

### Console testing

```ruby
User.all                    # => []
u = User.new                # unsaved user
u.valid?                    # => false (after validations added)
u.errors.full_messages      # friendly error messages
u3 = User.new(username: "alice")
u3.valid?                   # => true
u3.save                     # first user saved
```

---

## 📝 Post Model

### Generate the model

```bash
rails generate model Post title:string body:text user:references
rails db:migrate
```

### Validations added in `app/models/post.rb`

```ruby
class Post < ApplicationRecord
  belongs_to :user
  has_many :comments

  validates :title, presence: true
  validates :body, presence: true
  validates :user_id, presence: true
end
```

### Console testing

```ruby
User.first.posts                # => [] (no posts yet, no error)
p1 = Post.new(title: "Hello", body: "First post", user_id: 1)
p2 = User.first.posts.build     # user_id auto-filled
p1.save
Post.first.user                 # => returns the User object
```

---

## 💬 Comment Model

### Generate the model

```bash
rails generate model Comment body:text user:references post:references
rails db:migrate
```

### Validations added in `app/models/comment.rb`

ruby
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :post

  validates :body, presence: true
  validates :user_id, presence: true
  validates :post_id, presence: true
end
```

### Associations recap

```ruby
# app/models/user.rb
has_many :posts
has_many :comments

# app/models/post.rb
belongs_to :user
has_many :comments

# app/models/comment.rb
belongs_to :user
belongs_to :post
```

### Console testing

```ruby
u2 = User.find(2)
c1 = u2.comments.first          # user's comment
c1.user                         # => u2

p1 = Post.first
p1.comments.first               # => c1
c1.post                         # => p1
```

---

## ✅ Final Data Model

| Model   | Columns                              | Associations                     |
|---------|--------------------------------------|----------------------------------|
| User    | `username`                           | has_many :posts, :comments       |
| Post    | `title`, `body`, `user_id`           | belongs_to :user, has_many :comments |
| Comment | `body`, `user_id`, `post_id`         | belongs_to :user, :post          |

---

## 🧪 Helpful Console Commands

```ruby
reload!                # reload console after code changes
quit                   # exit console
User.new               # build unsaved record
User.create(...)       # build and save
u.valid?               # run validations
u.errors.full_messages # see errors
User.first.posts.build # build via association
```

---

## 📌 Notes

- No authentication or password security is implemented.
- Comments belong to a `Post` and a `User` — users cannot comment on comments.
- All foreign keys are required to prevent orphan records.