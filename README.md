# CASE-STUDY-PREDICTIVE-ANALYTICS-IN-HOTEL-BOOKINGMANAGEMENT
This repository contains a comprehensive data science project aimed at predicting customer behaviors and booking cancellations in the hotel industry. The project is designed to provide practical experience in key areas of data science, including Exploratory Data Analysis (EDA), model implementation, and handling class imbalances.


import pandas as pd
# Load the dataset
file_path = '/content/Hotel Reservations (1).csv'
df = pd.read_csv(file_path)
# Display the first few rows of the dataframe
df.head()

**Data Cleaning and Preprocessing: Checking the anomalies and missing values in the dataset.**
# Checking for missing values
missing_values = df.isnull().sum()

# Checking for any obvious anomalies in the dataset
# This includes checking for negative values in columns where it doesn't make sense (e.g., number of adults, children, etc.)
anomalies = {}
for column in df.columns:
    if df[column].dtype in ['int64', 'float64']:
        anomalies[column] = df[df[column] < 0][column].count()

missing_values, anomalies

**Duplicates**
# Identifying duplicate records in the dataset
duplicates = df[df.duplicated()]

# Counting the number of duplicate records
num_duplicates = duplicates.shape[0]

# Removing the duplicates from the original dataframe
df_cleaned = df.drop_duplicates()

num_duplicates, df_cleaned.shape

**Correcting errors**
# Checking for anomalies in numerical columns
numerical_columns = df_cleaned.select_dtypes(include=['int64', 'float64']).columns
numerical_anomalies = {col: {"min": df_cleaned[col].min(), "max": df_cleaned[col].max()} for col in numerical_columns}

# Checking for inconsistencies in categorical columns
categorical_columns = df_cleaned.select_dtypes(include=['object']).columns
categorical_inconsistencies = {col: df_cleaned[col].unique() for col in categorical_columns if col != 'Booking_ID'}

# Checking date-related columns for plausibility
# 'arrival_year', 'arrival_month', and 'arrival_date' are checked for plausible ranges
date_anomalies = {
    "arrival_year": {"min": df_cleaned['arrival_year'].min(), "max": df_cleaned['arrival_year'].max()},
    "arrival_month": {"min": df_cleaned['arrival_month'].min(), "max": df_cleaned['arrival_month'].max()},
    "arrival_date": {"min": df_cleaned['arrival_date'].min(), "max": df_cleaned['arrival_date'].max()}
}

numerical_anomalies, categorical_inconsistencies, date_anomalies

**Handling Outliers**
# Capping outliers using the 1st and 99th percentiles
cols_to_cap = ['no_of_children', 'no_of_week_nights', 'lead_time',
               'avg_price_per_room', 'no_of_previous_cancellations',
               'no_of_previous_bookings_not_canceled']

for col in cols_to_cap:
    # Calculating the 1st and 99th percentiles
    lower_bound = df_cleaned[col].quantile(0.01)
    upper_bound = df_cleaned[col].quantile(0.99)

    # Capping the outliers
    df_cleaned[col] = df_cleaned[col].clip(lower=lower_bound, upper=upper_bound)

# Plotting boxplots again to visualize the effect of capping
plt.figure(figsize=(15, 10))
for i, col in enumerate(cols_to_cap, 1):
    plt.subplot(3, 2, i)
    sns.boxplot(y=df_cleaned[col])
    plt.title(col)

plt.tight_layout()
plt.show()

**Statistical summary**
# Calculating the statistical summary of the numerical attributes
statistical_summary = df_cleaned.describe()

# Including skewness and kurtosis for understanding the shape of the distribution
skewness = df_cleaned.skew().rename('skewness')
kurtosis = df_cleaned.kurtosis().rename('kurtosis')

# Appending these to the statistical summary
statistical_summary = statistical_summary.append([skewness, kurtosis])

statistical_summary

**Data Visualization**
# Histograms for key numerical variables
num_cols_histograms = ['no_of_adults', 'no_of_children', 'no_of_weekend_nights',
                       'no_of_week_nights', 'avg_price_per_room', 'lead_time',
                       'no_of_special_requests']

plt.figure(figsize=(15, 10))
for i, col in enumerate(num_cols_histograms, 1):
    plt.subplot(3, 3, i)
    sns.histplot(df_cleaned[col], kde=True)
    plt.title(col)

plt.tight_layout()
plt.show()

# Box plots for the same variables
plt.figure(figsize=(15, 10))
for i, col in enumerate(num_cols_histograms, 1):
    plt.subplot(3, 3, i)
    sns.boxplot(x=df_cleaned[col])
    plt.title(col)

plt.tight_layout()
plt.show()

# Scatter plots for selected pairs of variables
scatter_pairs = [('avg_price_per_room', 'no_of_adults'),
                 ('lead_time', 'avg_price_per_room'),
                 ('no_of_week_nights', 'no_of_weekend_nights')]

plt.figure(figsize=(15, 5))
for i, pair in enumerate(scatter_pairs, 1):
    plt.subplot(1, 3, i)
    sns.scatterplot(data=df_cleaned, x=pair[0], y=pair[1])
    plt.title(f'{pair[0]} vs. {pair[1]}')

plt.tight_layout()
plt.show()

**Correlation matrix**
# Encoding the 'booking_status' column: 'Canceled' = 1, 'Not_Canceled' = 0
df_cleaned['booking_status_encoded'] = df_cleaned['booking_status'].apply(lambda x: 1 if x == 'Canceled' else 0)

# Calculating the correlation matrix
correlation_matrix = df_cleaned.corr()

# Focusing on the correlation of variables with the target variable 'booking_status_encoded'
correlation_with_target = correlation_matrix['booking_status_encoded'].sort_values(ascending=False)

correlation_with_target

# Generating a heatmap for the correlation matrix
plt.figure(figsize=(12, 8))
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', fmt=".2f")
plt.title("Correlation Heatmap")
plt.show()

**Feature Engineering:**
Encoding
# One-hot encoding for 'room_type_reserved' and 'type_of_meal_plan'
room_type_dummies = pd.get_dummies(df_cleaned['room_type_reserved'], prefix='room_type')
meal_plan_dummies = pd.get_dummies(df_cleaned['type_of_meal_plan'], prefix='meal_plan')

# Concatenating the new dummy variables with the original dataframe
df_encoded = pd.concat([df_cleaned, room_type_dummies, meal_plan_dummies], axis=1)

# Dropping the original categorical columns
df_encoded.drop(['room_type_reserved', 'type_of_meal_plan'], axis=1, inplace=True)

# Displaying the first few rows of the updated dataframe
df_encoded.head()

**Creating new features**

df_encoded['total_stay_duration'] = df_encoded['no_of_weekend_nights'] + df_encoded['no_of_week_nights']
df_encoded['total_guests'] = df_encoded['no_of_adults'] + df_encoded['no_of_children']
df_encoded['lead_time_category'] = pd.cut(df_encoded['lead_time'], bins=[0, 30, 90, float('inf')], labels=['Short', 'Medium', 'Long'])

# Room to Guest Ratio (assuming each booking is for one room)
df_encoded['room_to_guest_ratio'] = 1 / df_encoded['total_guests']
df_encoded['room_to_guest_ratio'].replace(float('inf'), 0, inplace=True)  # Replacing division by zero with 0

# Displaying the first few rows with new features
df_encoded[['total_stay_duration', 'total_guests', 'lead_time_category', 'room_to_guest_ratio']].head()

**feature selection and reducing dimensionality**
print(X.dtypes)
categorical_columns = ['market_segment_type']  # Replace with your actual categorical column names
X_encoded = pd.get_dummies(X, columns=categorical_columns)
from sklearn.ensemble import RandomForestClassifier

forest = RandomForestClassifier(n_estimators=100, random_state=42, n_jobs=-1)
forest.fit(X_encoded, y)

# Getting feature importances
importances = forest.feature_importances_
feature_names = X_encoded.columns
feature_importance_df = pd.DataFrame({'Feature': feature_names, 'Importance': importances})
print(feature_importance_df.sort_values(by='Importance', ascending=False))
from sklearn.feature_selection import SelectFromModel

selector = SelectFromModel(forest, prefit=True)
selected_features = feature_names[selector.get_support()]

print(selected_features)
# Encoding remaining categorical variables using one-hot encoding
market_segment_dummies = pd.get_dummies(df_encoded['market_segment_type'], prefix='market_segment')

# Concatenating the new dummy variables with the original dataframe
df_encoded_all = pd.concat([df_encoded, market_segment_dummies], axis=1)

# Dropping the original categorical columns
df_encoded_all.drop(['market_segment_type'], axis=1, inplace=True)

# Preparing the data for feature selection again
X_all = df_encoded_all.drop(['Booking_ID', 'booking_status', 'booking_status_encoded', 'lead_time_category'], axis=1)

# Using RandomForestClassifier to determine feature importance
forest_all = RandomForestClassifier(n_estimators=100, random_state=42, n_jobs=-1)
forest_all.fit(X_all, y)

# Getting feature importances
importances_all = forest_all.feature_importances_
feature_names_all = X_all.columns
feature_importance_df_all = pd.DataFrame({'Feature': feature_names_all, 'Importance': importances_all})

# Selecting features based on importance
selector_all = SelectFromModel(forest_all, prefit=True)
selected_features_all = feature_names_all[selector_all.get_support()]

selected_features_all, feature_importance_df_all.sort_values(by='Importance', ascending=False)

**Scaling**
from sklearn.preprocessing import StandardScaler

# Selecting the features identified as important
features_to_scale = df_encoded_all[selected_features_all]

# Initializing the StandardScaler
scaler = StandardScaler()

# Scaling the selected features
scaled_features = scaler.fit_transform(features_to_scale)

# Creating a DataFrame for the scaled features
scaled_features_df = pd.DataFrame(scaled_features, columns=selected_features_all)

# Displaying the first few rows of the scaled features DataFrame
scaled_features_df.head()

**Splitting the dataset**
from sklearn.model_selection import train_test_split

# Splitting the dataset into training and testing sets with a 70-30 split
X_train, X_test, y_train, y_test = train_test_split(scaled_features_df, y, test_size=0.3, random_state=42)

# Displaying the shapes of the training and testing sets
X_train.shape, X_test.shape, y_train.shape, y_test.shape

**Model Selection**
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

# Training a Decision Tree model
dt_classifier = DecisionTreeClassifier(random_state=42)
dt_classifier.fit(X_train, y_train)

# Predicting on the test set
y_pred = dt_classifier.predict(X_test)

# Evaluating the model
accuracy = accuracy_score(y_test, y_pred)
report = classification_report(y_test, y_pred)
conf_matrix = confusion_matrix(y_test, y_pred)

accuracy, report, conf_matrix

**Training Models: Train models like KNN and Decision Trees on the training set.**
from sklearn.neighbors import KNeighborsClassifier

# Training a KNN model
knn_classifier = KNeighborsClassifier(n_neighbors=5)  # using default k=5
knn_classifier.fit(X_train, y_train)

# Predicting on the test set
y_pred_knn = knn_classifier.predict(X_test)

# Evaluating the KNN model
accuracy_knn = accuracy_score(y_test, y_pred_knn)
report_knn = classification_report(y_test, y_pred_knn)
conf_matrix_knn = confusion_matrix(y_test, y_pred_knn)

accuracy_knn, report_knn, conf_matrix_knn

**hyperparameter tuning**
from sklearn.model_selection import GridSearchCV

# Setting up parameter grid for the Decision Tree
param_grid_dt = {
    'max_depth': [10, 15, 20, None],
    'min_samples_leaf': [1, 2, 4],
    'min_samples_split': [2, 5, 10]
}

# Grid Search for Decision Tree
grid_search_dt = GridSearchCV(estimator=DecisionTreeClassifier(random_state=42),
                              param_grid=param_grid_dt,
                              cv=5,
                              n_jobs=-1,
                              verbose=1)

grid_search_dt.fit(X_train, y_train)
best_dt = grid_search_dt.best_estimator_

# Setting up parameter grid for KNN
param_grid_knn = {
    'n_neighbors': [3, 5, 7, 9],
    'weights': ['uniform', 'distance']
}

# Grid Search for KNN
grid_search_knn = GridSearchCV(estimator=KNeighborsClassifier(),
                               param_grid=param_grid_knn,
                               cv=5,
                               n_jobs=-1,
                               verbose=1)

grid_search_knn.fit(X_train, y_train)
best_knn = grid_search_knn.best_estimator_

# Evaluating the tuned models
y_pred_best_dt = best_dt.predict(X_test)
y_pred_best_knn = best_knn.predict(X_test)

accuracy_best_dt = accuracy_score(y_test, y_pred_best_dt)
accuracy_best_knn = accuracy_score(y_test, y_pred_best_knn)

best_dt_params = grid_search_dt.best_params_
best_knn_params = grid_search_knn

**SMOTE**
from imblearn.over_sampling import SMOTE

# Applying SMOTE to the training data
smote = SMOTE(random_state=42)
X_train_smote, y_train_smote = smote.fit_resample(X_train, y_train)

# Checking the new class distribution after applying SMOTE
new_class_distribution = pd.Series(y_train_smote).value_counts()
new_class_distribution_percentage = pd.Series(y_train_smote).value_counts(normalize=True) * 100

# Retraining the Decision Tree model on the balanced dataset
dt_classifier_smote = DecisionTreeClassifier(random_state=42)
dt_classifier_smote.fit(X_train_smote, y_train_smote)
y_pred_dt_smote = dt_classifier_smote.predict(X_test)
accuracy_dt_smote = accuracy_score(y_test, y_pred_dt_smote)

# Retraining the KNN model on the balanced dataset
knn_classifier_smote = KNeighborsClassifier(n_neighbors=5)
knn_classifier_smote.fit(X_train_smote, y_train_smote)
y_pred_knn_smote = knn_classifier_smote.predict(X_test)
accuracy_knn_smote = accuracy_score(y_test, y_pred_knn_smote)

new_class_distribution, new_class_distribution_percentage, accuracy_dt_smote, accuracy_knn_smote

**Visualization 1: Distribution of Room Types Reserved**
import matplotlib.pyplot as plt
import seaborn as sns
plt.figure(figsize=(10, 6))
sns.countplot(data=df, x='room_type_reserved')
plt.title('Distribution of Room Types Reserved')
plt.xticks(rotation=45)
plt.show()

**Question 2: How does the average price per room vary by room type?**
avg_price_by_room_type = df.groupby('room_type_reserved')['avg_price_per_room'].mean().sort_values()
plt.figure(figsize=(10, 6))
sns.barplot(x=avg_price_by_room_type.index, y=avg_price_by_room_type.values)
plt.title('Average Price Per Room by Room Type')
plt.xticks(rotation=45)
plt.show()

**Question 3: What is the trend of bookings over different months of the year?**
bookings_by_month = df['arrival_month'].value_counts().sort_index()
plt.figure(figsize=(10, 6))
sns.lineplot(x=bookings_by_month.index, y=bookings_by_month.values)
plt.title('Trend of Bookings Over Different Months')
plt.xlabel('Month')
plt.ylabel('Number of Bookings')
plt.show()

**Question4: What is the relationship between the type of meal plan and the average price per room?**
plt.figure(figsize=(10, 6))
sns.boxplot(data=df, x='type_of_meal_plan', y='avg_price_per_room')
plt.title('Average Price Per Room by Meal Plan Type')
plt.xticks(rotation=45)
plt.show()

**Question5: How does the requirement for car parking space relate to the number of adults in a booking?**
# First, create a grouped data for the heatmap
parking_grouped = df.groupby(['required_car_parking_space', 'no_of_adults']).size().unstack(fill_value=0)

plt.figure(figsize=(10, 6))
sns.heatmap(parking_grouped, annot=True, fmt="d", cmap="YlGnBu")
plt.title('Correlation between Number of Adults and Car Parking Space Requirements')
plt.ylabel('Required Car Parking Space')
plt.xlabel('Number of Adults')
plt.show()

**Question6: What is the distribution of lead time for bookings, and how does it vary by booking status (canceled vs. not canceled)?
**
plt.figure(figsize=(10, 6))
sns.histplot(data=df, x='lead_time', hue='booking_status', element='step', stat='density', common_norm=False)
plt.title('Distribution of Lead Time by Booking Status')
plt.xlabel('Lead Time')
plt.ylabel('Density')
plt.show()

**Question7: Do certain months of the year have a higher number of special requests?**
special_requests_by_month = df.groupby('arrival_month')['no_of_special_requests'].sum()

plt.figure(figsize=(10, 6))
sns.barplot(x=special_requests_by_month.index, y=special_requests_by_month.values)
plt.title('Total Special Requests per Month')
plt.xlabel('Month')
plt.ylabel('Total Special Requests')
plt.show()














