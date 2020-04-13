Here is a step-by-step procedure that I used to configure my Slack App integration with Django and ngrok

**IMPORTANT**: all of the logic that gives my phrasefinder slackbot functionality can be found in slack/events/views.py

--Install frameworks and slack api----------------------------------------------
pip install django
pip install djangorestframework
pip install slackclient==1.3.2
pip install slackeventsapi

--Create our Django application-------------------------------------------------
django-admin startproject slack
cd slack
django-admin startapp events

--Add 'rest_framework' and 'events' to INSTALLED_APPS in slack/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'events',
]

--Export Slack Bot Bot User OAuth Access Token (can be found on Installed App Settings)

export SLACK_BOT_TOKEN=xoxb-1047294149206-1053947245045-RQxdqXY1R0mSA6UiLnK2HiG9

--Add SLACK API App keys to slack/settings.py-----------------------------------

# SLACK API Configurations
# ----------------------------------------------
# PhraseFinder Keys
SLACK_CLIENT_ID = '1047294149206.1039384436067'
SLACK_CLIENT_SECRET = '850d7ce3fa04c5183aed822073819163'
SLACK_VERIFICATION_TOKEN = '7TMy52dmieCxugnKQuThKBlI'
SLACK_BOT_USER_TOKEN = 'xoxb-1047294149206-1053947245045-RQxdqXY1R0mSA6UiLnK2HiG9'

--Apply all migrations and run the Django server--------------------------------

python manage.py migrate
python manage.py runserver

Command line should give you: Starting development server at http://'my-ip-address'/
or whatever HTTP URL it creates

--Create our API endpoint with the Django REST framework in events/views.py-----

from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.conf import settings

SLACK_VERIFICATION_TOKEN = getattr(settings, 'SLACK_VERIFICATION_TOKEN', None)

class Events(APIView):
    def post(self, request, *args, **kwargs):
        slack_message = request.data
        if slack_message.get('token') != SLACK_VERIFICATION_TOKEN:
            return Response(status=status.HTTP_403_FORBIDDEN)
        return Response(status=status.HTTP_200_OK)

--Tie together the Events API to our server url: http://'my-ip-address'/events/--

Within slack/urls.py:

from django.conf.urls import url
from django.contrib import admin
from django.urls import path
from events.views import Events

urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^events/', Events.as_view()),
]

--Add in verification logic so that Slack can verify our endpoint---------------

Within events.views.py in the Events class:

# verification
    if slack_message.get('type') == 'url_verification':
        return Response(data=slack_message,
                        status=status.HTTP_200_OK)
