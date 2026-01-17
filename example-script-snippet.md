public class TestComponent extends EntityEventSystem<EntityStore, BreakBlockEvent> {


    protected TestComponent() {
        super(BreakBlockEvent.class);
    }

    @Override
    public void handle(int i, @NotNull ArchetypeChunk<EntityStore> archetypeChunk, @NotNull Store<EntityStore> store, @NotNull CommandBuffer<EntityStore> commandBuffer, @NotNull BreakBlockEvent breakBlockEvent) {
        System.out.println("BLOCK BREAK");
        System.out.println(store.getClass());
        PlayerRef playerRef = archetypeChunk.getComponent(i, PlayerRef.getComponentType());

        System.out.println("Player: " + playerRef.getUsername());
    }

    @Nullable
    @Override
    public Query<EntityStore> getQuery() {
        return PlayerRef.getComponentType();
    }
}