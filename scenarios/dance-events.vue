<template>
  <Scenarios id="dance-events" title="Dance Events">
    <Scenario id="welcome" title="Welcome">
      <Var v-model="$ConversationsCount" :val="$ConversationsCount || 0"/>
      <Var v-model="$GreetingText" :request="Quotes.getRandom({ type: 'greeting', name: $MyProfile.name })"/>
      <Text>
        {{ $GreetingText }}.
        How can I serve you?
      </Text>

      <Play v-if="$ConversationsCount === 0" id="dance-now" />
      <Var v-else v-model="$Input" userInput @input="Stop()" then="FindScenarios($Input)" />
    </Scenario>

    <Scenario id="dance-now" title="Where can I dance today?">
      <Var v-model="$Type" value="events"/>
      <Var v-model="$MyCity" :value="$MyProfile.currentCity"/>
      <Var v-model="$Date" value="today"/>
      <Var v-model="$MyGenres" :value="$MyProfile.genres"/>
      
      <Var v-model="$FoundEvents" request="Search($Type, $MyCity, $Date, $MyGenres)" then="events-found" empty="events-notfound"/>
    </Scenario>

    <Scenario id="events-found">
      <Var v-model="$FoundGenres" request="GetAllGenres($FoundEvents)" then="genres-found" empty="genres-notfound"/>
    </Scenario>
    <Scenario id="genres-found">
      <Text>
        What would you like to dance today?
      </Text>
      <Var v-model="$SelectedGenre" userInput :options="$FoundGenres" then="events-results"/>
    </Scenario>
    <Scenario id="events-results">
      <Var v-model="$FilteredEvents" request="Filter($FoundEvents, { genre: $SelectedGenre })"/>
      <Var v-model="$SelectedResultIndex" :val="$SelectedResultIndex || 0"/>
      <Var v-model="$SelectedEvent" :val="$FilteredEvents[$selectedResult]"/>
      
      <Text>
        There is a {{ $SelectedEvent.name }} near {{ $SelectedEvent.subwayStation }} with {{ $SelectedEvent.guests | count }} guests.
        Wanna go?
      </Text>

      <Event id="$SelectedEvent.id"/>

      <Var v-model="$RSVP" userInput then="rsvp">
        <Else>
          <Var v-model="$SelectedResultIndex" :val="$SelectedResultIndex + 1"/>
          <Play id="events-results" />
        </Else>
      </Var>
    </Scenario>
    <Scenario id="rsvp">
      <Var v-model="$BookingStatus" :request="Event.RSVP($SelectedEvent, 'yes')" then="rsvp-confirmed" else="rsvp-aborted"/>
    </Scenario>
    <Scenario id="rsvp-aborted">
      <Text>
        There was a problew with your RSVP: {{ $BookingStatus.error }}.
        Would you like to try again?
        <Var v-model="$RSVP" userInput then="rsvp">
          <Else>
            <Var v-model="$SelectedResultIndex" :val="$SelectedResultIndex + 1"/>
            <Play id="events-results" />
        </Else>
        </Var>
      </Text>
    </Scenario>
    <Scenario id="rsvp-confirmed">
      <Text>
        Booking is confirmed today at {{ $SelectedEvent.startAt | time }}
        <Play id="end" />
      </Text>
    </Scenario>
  </Scenarios>
</template>
