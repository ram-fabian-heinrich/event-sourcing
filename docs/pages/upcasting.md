Corrected version (spelling, grammar, and clarity only):

⸻

Upcasting

There are cases where we already have events in our stream, but some data is missing or not in the correct format for a new use case. Normally, you would need to create versioned events for this. This can lead to many versions of the same event, which can cause unnecessary complexity.

To prevent this, we offer an Upcaster, which can operate on the payload before it is denormalized into an event object. There, you can change the event name and adjust the event payload.

Adjust payload

Assume we have a ProfileCreated event that contains an email address. The business now requires all email addresses to be stored in lowercase.

One option would be to adjust the aggregate and the projections to handle this. Alternatively, this can be done beforehand, so there is no need to maintain the same logic in multiple places.

use Patchlevel\EventSourcing\Serializer\Upcast\Upcast;
use Patchlevel\EventSourcing\Serializer\Upcast\Upcaster;

final class ProfileCreatedEmailLowerCastUpcaster implements Upcaster
{
    public function __invoke(Upcast $upcast): Upcast
    {
        // Ignore if a different event is processed
        if ($upcast->eventName !== 'profile.created') {
            return $upcast;
        }

        if (!array_key_exists('email', $upcast->payload) || !is_string($upcast->payload['email'])) {
            return $upcast;
        }

        return $upcast->replacePayloadByKey('email', strtolower($upcast->payload['email']));
    }
}

!!! warning

Other events are also passed to the upcaster. An early return is therefore recommended.

Adjust event name

Sometimes an event name turns out to be a poor choice and needs to be changed. An Upcaster can be used to rename events.

use Patchlevel\EventSourcing\Serializer\Upcast\Upcast;
use Patchlevel\EventSourcing\Serializer\Upcast\Upcaster;

final class EventNameRenameUpcaster implements Upcaster
{
    /** @param array<string, string> $eventNameMap */
    public function __construct(
        private readonly array $eventNameMap,
    ) {
    }

    public function __invoke(Upcast $upcast): Upcast
    {
        if (array_key_exists($upcast->eventName, $this->eventNameMap)) {
            return $upcast->replaceEventName($this->eventNameMap[$upcast->eventName]);
        }

        return $upcast;
    }
}

!!! tip

Events can also have [aliases](./events.md#alias). This is usually sufficient.

Configure

After defining the upcasting rules, they must be passed to the serializer. Since multiple upcasters are involved, a chain is used.

use Patchlevel\EventSourcing\Metadata\Event\EventRegistry;
use Patchlevel\EventSourcing\Serializer\DefaultEventSerializer;
use Patchlevel\EventSourcing\Serializer\Upcast\UpcasterChain;

/** @var EventRegistry $eventRegistry */
$upcaster = new UpcasterChain([
    new ProfileCreatedEmailLowerCastUpcaster(),
    new EventNameRenameUpcaster(['old_event_name' => 'new_event_name']),
]);

$serializer = DefaultEventSerializer::createFromPaths(
    ['src/Domain'],
    $upcaster,
);

Learn more
	•	How to create messages￼
	•	How to define events￼
	•	How to configure the store￼