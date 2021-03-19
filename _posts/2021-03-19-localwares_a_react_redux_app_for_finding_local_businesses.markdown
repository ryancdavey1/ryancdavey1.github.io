---
layout: post
title:      "LocalWares: A React/Redux app for finding local businesses"
date:       2021-03-19 08:21:25 +0000
permalink:  localwares_a_react_redux_app_for_finding_local_businesses
---


## Background
When visiting a new city, I often have the urge to find the coolest local shops within walking distance, which is why I created LocalWares. Users can sign up or log in, and create businesses with various details.

I decided to follow the process from this [article](https://agiletkiewicz.github.io/successfully_planning_projects).
For user stories, it felt necessary to guarantee that: users could see a list of businesses, those businesses could list items/services, and that each business could fit into a definable category. As far as stretch goals, I included distance filtering, etc. users should only be able to edit businesses that they've made, if edit capability is available, which is why I've left editing last. The subtasks component felt straightforward after fleshing out the user stories - although I realized I may end up modifying the position of the favorites and search (would they be separate routes that could display on a separate page?).

## API Schema and Models 

Laying out the schema became more complicated with deciding on locating a business; users could simply type in an address, which is then saved as text. But what if I reached that stretch goal of doing distance based filtering? For this, I turned to the geokit gem, which would allow me to turn address inputs into coordinates. 

```
From db/schema.rb:

  create_table "users", force: :cascade do |t|
    t.string "name"
    t.string "email"
    t.string "password_digest"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

create_table "businesses", force: :cascade do |t|
    t.string "name"
    t.string "description"
    t.string "open_hours"
    t.string "email"
    t.string "phone_number"
    t.boolean "favorite"
    t.boolean "delivery"
    t.integer "user_id"
    t.integer "category_id"
    t.string "website"
    t.string "address1"
    t.string "city"
    t.string "state"
    t.string "postal_code"
    t.float "lat"
    t.float "lng"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end
```

It felt complicated to leave the Business model with a street address, city, postal code, and then calculated latitude and longitude, so I created a separate location model, that contained all of that information, and then connected both Business and Location model with a location_id and business_id respectively. 
That didn't quite work, so I turned to the geocoder gem, and tested it out with a sample Location (include test.js file); I then switched back to including the coordinates with the Business class, I realized I wasnt going to be using a location separately from a business at any time - the location filtering would always be centered around the coordinates attached to the business, and the "location" wouldn't be used on its own.
The `geocoded_by` atttributes assigns the result of geocoding a businesses address (which is assembled in the address method) to latitude and longitude variables, named `lat` and `lng`.

```
class Business < ApplicationRecord
  belongs_to :user
  belongs_to :category
  has_many :items

  validates :name, :description, :user, :category_id, :address1, :city, :state, :postal_code, presence: true

  geocoded_by :address, :latitude => :lat, :longitude => :lng
  after_validation :geocode#, :if => lambda{ |obj| obj.address1_changed? || obj.city_changed? || obj.state_changed? || obj.postal_code_changed? }

  def address
    #[state, city, postal_code, address1].compact.join(', ')
    [address1, city, state, postal_code].compact.join(', ')
  end
end
```

Next I focused on serializing user data - making sure with the show/index routes that only name and email were returning in the json:

```

class UserSerializer
  include JSONAPI::Serializer
  attributes :name, :email
end


# GET /users
  def index
    @users = User.all

    #json_string = MovieSerializer.new(movie).serializable_hash.to_json
    users_json = UserSerializer.new(@users).serializable_hash.to_json
    render json: users_json
  end

  # POST /users
  def create
    @user = User.new(user_params)
    
    if @user.save
      session[:user_id] = @user.id
      render json: UserSerializer.new(@user), status: :created
    else
      resp = {
        error: @user.errors.full_messages.to_sentence
      }
      render json: resp, status: :unprocessable_entity
    end
  end
```

The serializing the business data also followed a similar process for the attributes of a business, although with more attribute options. I also opted 


## Adding Redux to the Front End
Next came the scaffold for the frontend project - `create-react-app`. I immediately went about creating a redux store, and reducers for all of the different models that I had defined on the backend: category, user, item, and business - currently the business and user reducers are used, but items and categories do not have CRUD functionality, and are not utilized. 

I added currentUser as a reducer for log in functionality:
```
const currentUserReducer = (state = null, action) => {
  switch(action.type) {
    case "SET_CURRENT_USER":
      return action.user
    case "CLEAR_CURRENT_USER":
      return null
    default:
      return state
  }
}

export default currentUserReducer;
```

All of the reducers were then combined in `store.js`, which was the `store` used in `index.js`. The index file used `Provider` from the `react-redux` npm package to add the store to the app. `BrowserRouter` also would allow for routing between different pages.

```

const reducer = combineReducers({
  users: usersReducer,
  currentUser: currentUserReducer,
  loginForm: loginFormReducer,
  businesses: businessesReducer,
  signupForm: signupFormReducer,
  businessForm: businessFormReducer,
  categories: categoriesReducer
})

const composeEnhancer = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose; 

const store = createStore(reducer, composeEnhancer(applyMiddleware(thunk)))

export default store;
```


```
From index.js

ReactDOM.render(
  <Provider store={store}>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </Provider>,
  document.getElementById('root')
);
```


## User Sign In
For the logging in aspect, I would need to create a sessions controller on the backend. For this controller, I needed to add create and destroy session methods to set or remove the user id within the session object, and a current user method to get user information if logged in correctly.

```
class Api::V1::SessionsController < ApplicationController
  def create
    @user = User.find_by(email: params[:session][:email])

    if @user && @user.authenticate(params[:session][:password])
      session[:user_id] = @user.id
      render json: UserSerializer.new(current_user), status: :ok
    else
      render json: {
        error: "Invalid email or password"
      }
    end
  end

  def get_current_user
    if logged_in?
      render json: UserSerializer.new(current_user)
    else
      render json: {
        error: "Not currently logged in"
      }
    end
  end

  def destroy
    session.clear
    render json: {
      notice: "successfully logged out"
    }, status: :ok
  end

end
```

## Form Components

Next, I made a new  business form: it has all of the necessary details for creating a business. The `UpdateBusinessForm` action came from a separate actions file for businesses, and `connect` from the `react-redux` npm module allowed the form to be connected to the store. 

```
const mapStateToProps = state => {
  const userId = state.currentUser ? state.currentUser.id : ""
  return {
    formData: state.businessForm,
    userId
  }
}


export default connect(mapStateToProps, { updateBusinessForm })(BusinessForm);
```

I did try to add categories as a dropdown, and attempted to get categories from the backend with componentDidMount- however because I have a limited list of categories, that the user can't add to at the moment, it felt a bit overcomplicated, so I just used a basic radio button selection for the category id.

At first I did have issues trying to get the business to save, I made a few errors with getting the action and value to update properly in the store;

I did try to switch the form to a react component that relied on local state; this did work to update the business details;

However, knowing that I would add edit and delete functionality for individual businesses, I switched the form back to being connected to the redux store, and managed to get the submit function to work properly and update the store.


Edit funcitonality did not display until I changed all of the newbusinessform ->businessform labels so that I could keep consistent
Switched category back again to using handleChange and having default check for the category 

this is where I ran into a problem - because I had added edit action/case in my reducer, the fields for the business form were now being set to the business that was most recently being edited.
