# keloysiusmak
###### \java\seedu\address\logic\commands\CommandTestUtil.java
``` java

    /**
     * Executes the given {@code command}, confirms that <br>
     * - the result message matches {@code expectedMessage} <br>
     * - the {@code actualConfig} matches {@code expectedConfig}
     */
    public static void assertConfigCommandSuccess(Command command, Config actualConfig, String expectedMessage,
                                                  Config expectedConfig) {
        try {
            CommandResult result = command.execute();
            assertEquals(expectedMessage, result.feedbackToUser);
            assertEquals(expectedConfig, actualConfig);
        } catch (CommandException ce) {
            throw new AssertionError("Execution of command should not fail.", ce);
        } catch (DuplicateUserException due) {
            throw new AssertionError("Execution of command should not fail.", due);
        }
    }

    /**
     * Executes the given {@code command}, confirms that <br>
     * - the result message matches {@code expectedMessage} <br>
     * - the {@code actualConfig} does not match {@code expectedConfig}
     */
    public static void assertConfigDiffCommandSuccess(Command command, Config actualConfig, String expectedMessage,
                                                      Config expectedConfig) {
        try {
            CommandResult result = command.execute();
            assertEquals(expectedMessage, result.feedbackToUser);
            assertNotEquals(expectedConfig, actualConfig);
        } catch (CommandException ce) {
            throw new AssertionError("Execution of command should not fail.", ce);
        } catch (DuplicateUserException due) {
            throw new AssertionError("Execution of command should not fail.", due);
        }
    }
```
###### \java\seedu\address\logic\commands\SetAliasCommandTest.java
``` java
package seedu.address.logic.commands;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;
import static org.junit.Assert.fail;
import static seedu.address.commons.core.Messages.MESSAGE_DUPLICATE_ALIAS;
import static seedu.address.commons.core.Messages.MESSAGE_UNKNOWN_COMMAND;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.function.Predicate;

import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ExpectedException;

import javafx.collections.ObservableList;
import seedu.address.commons.core.Config;
import seedu.address.logic.CommandHistory;
import seedu.address.logic.UndoRedoStack;
import seedu.address.logic.commands.exceptions.CommandException;
import seedu.address.model.Model;
import seedu.address.model.ReadOnlyAccount;
import seedu.address.model.ReadOnlyAddressBook;
import seedu.address.model.ReadOnlyEventBook;
import seedu.address.model.alias.Alias;
import seedu.address.model.alias.exceptions.DuplicateAliasException;
import seedu.address.model.alias.exceptions.UnknownCommandException;
import seedu.address.model.event.ReadOnlyEvent;
import seedu.address.model.event.exceptions.DuplicateEventException;
import seedu.address.model.event.exceptions.EventNotFoundException;
import seedu.address.model.group.Group;
import seedu.address.model.person.Person;
import seedu.address.model.person.ReadOnlyPerson;
import seedu.address.model.person.exceptions.DuplicatePersonException;
import seedu.address.model.person.exceptions.PersonNotFoundException;
import seedu.address.model.person.exceptions.UnrecognisedParameterException;
import seedu.address.model.tag.Tag;
import seedu.address.model.user.ReadOnlyUser;
import seedu.address.model.user.exceptions.DuplicateUserException;
import seedu.address.testutil.AliasBuilder;

public class SetAliasCommandTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    public void constructor_nullCommandAndAlias_throwsNullPointerException() {
        thrown.expect(NullPointerException.class);
        new SetAliasCommand(null, null);
    }

    @Test
    public void execute_aliasAcceptedByModel_addSuccessful() throws Exception {
        ModelStubAcceptingAliasSet modelStub = new ModelStubAcceptingAliasSet();
        Alias validAlias = new AliasBuilder().build();

        CommandResult commandResult = getSetAliasCommand(validAlias, modelStub).execute();

        assertEquals(String.format(SetAliasCommand.MESSAGE_SUCCESS, validAlias), commandResult.feedbackToUser);
        assertEquals(Arrays.asList(validAlias), modelStub.aliases);
    }

    @Test
    public void execute_throwsDuplicateAliasException() throws Exception {
        ModelStub modelStub = new ModelStubThrowingDuplicateAliasException();
        Alias validAlias = new AliasBuilder().build();

        thrown.expect(CommandException.class);
        thrown.expectMessage(MESSAGE_DUPLICATE_ALIAS);

        getSetAliasCommand(validAlias, modelStub).execute();
    }

    @Test
    public void execute_throwsUnknownCommandException() throws Exception {
        ModelStub modelStub = new ModelStubThrowingUnknownCommandException();
        Alias validAlias = new AliasBuilder().build();

        thrown.expect(CommandException.class);
        thrown.expectMessage(MESSAGE_UNKNOWN_COMMAND);

        getSetAliasCommand(validAlias, modelStub).execute();
    }

    @Test
    public void equals() {
        SetAliasCommand addAliceAlias = new SetAliasCommand("first", "alice");
        SetAliasCommand addBobAlias = new SetAliasCommand("first", "bob");

        // same object -> returns true
        assertTrue(addAliceAlias.equals(addAliceAlias));

        // same values -> returns true
        SetAliasCommand addAliceAliasCopy = new SetAliasCommand("first", "alice");
        assertTrue(addAliceAlias.equals(addAliceAliasCopy));

        // different types -> returns false
        assertFalse(addAliceAlias.equals(1));

        // null -> returns false
        assertFalse(addAliceAlias.equals(null));

        // different alias -> returns false
        assertFalse(addAliceAlias.equals(addBobAlias));
    }

    /**
     * Generates a new SetAliasCommand with the details of the given alias.
     */
    private SetAliasCommand getSetAliasCommand(Alias alias, Model model) {
        SetAliasCommand command = new SetAliasCommand(alias.getCommand(), alias.getAlias());
        command.setData(model, new CommandHistory(), new UndoRedoStack(), new Config());
        return command;
    }

    /**
     * A default model stub that have all of the methods failing.
     */
    private class ModelStub implements Model {
        @Override
        public void addPerson(ReadOnlyPerson person) throws DuplicatePersonException {
            fail("This method should not be called.");
        }

        @Override
        public void orderList(String parameter) {
            fail("This method should not be called.");
        }

        @Override
        public void resetData(ReadOnlyAddressBook newData) {
            fail("This method should not be called.");
        }

        @Override
        public ReadOnlyAddressBook getAddressBook() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public ReadOnlyAccount getAccount() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public void deletePerson(ReadOnlyPerson target) throws PersonNotFoundException {
            fail("This method should not be called.");
        }

        @Override
        public void updatePerson(ReadOnlyPerson target, ReadOnlyPerson editedPerson)
                throws DuplicatePersonException {
            fail("This method should not be called.");
        }

        @Override
        public void groupPerson(Person person, Group group) throws PersonNotFoundException {
            fail("This method should not be called.");
        }

        @Override
        public void deleteTag(Tag tag) throws PersonNotFoundException, DuplicatePersonException {
            fail("This method should not be called.");
        }

        @Override
        public ObservableList<ReadOnlyPerson> getFilteredPersonList() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public void updateFilteredListToShowAll() {
            fail("This method should not be called");
        }

        @Override
        public ArrayList<ArrayList<String>> getCommands() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public String getAliasForCommand(String commandName) {
            return null;
        }

        @Override
        public void updateFilteredPersonList(Predicate<ReadOnlyPerson> predicate) {
            fail("This method should not be called.");
        }

        @Override
        public void resetEventData(ReadOnlyEventBook newData) {

        }

        @Override
        public ReadOnlyEventBook getEventBook() {
            return null;
        }

        @Override
        public void deleteEvent(ReadOnlyEvent target) throws EventNotFoundException {

        }

        @Override
        public void addEvent(ReadOnlyEvent event) throws DuplicateEventException {

        }

        @Override
        public void updateEvent(ReadOnlyEvent target, ReadOnlyEvent editedEvent) throws DuplicateEventException,
                EventNotFoundException {

        }

        @Override
        public ObservableList<ReadOnlyEvent> getFilteredEventList() {
            return null;
        }

        @Override
        public void updateFilteredEventList(Predicate<ReadOnlyEvent> predicate) {

        }

        @Override
        public void orderEventList(String parameter) throws UnrecognisedParameterException {

        }

        @Override
        public void setAlias(String command, String alias) throws DuplicateAliasException, UnknownCommandException {

        }

        @Override
        public void persistUserAccount(ReadOnlyUser user) throws DuplicateUserException {

        }

        @Override
        public byte[] retrieveDigestFromStorage() {
            return new byte[0];
        }

        @Override
        public String retrieveSaltFromStorage(String userId) {
            return null;
        }
    }

    /**
     * A Model stub that always throw a DuplicateAliasException when trying to add a person.
     */
    private class ModelStubThrowingDuplicateAliasException extends ModelStub {

        public void setAlias(String command, String alias) throws DuplicateAliasException {
            throw new DuplicateAliasException(MESSAGE_DUPLICATE_ALIAS);
        }
    }

    /**
     * A Model stub that always throw a UnknownCommandException when trying to add a person.
     */
    private class ModelStubThrowingUnknownCommandException extends ModelStub {

        public void setAlias(String command, String alias) throws UnknownCommandException {
            throw new UnknownCommandException(MESSAGE_UNKNOWN_COMMAND);
        }
    }

    /**
     * A Model stub that always accept the alias being set.
     */
    private class ModelStubAcceptingAliasSet extends ModelStub {
        final ArrayList<Alias> aliases = new ArrayList<>();

        @Override
        public void setAlias(String command, String name) {
            aliases.add(new Alias(command, name));
        }
    }

}
```
###### \java\seedu\address\logic\commands\SetThemeCommandTest.java
``` java
package seedu.address.logic.commands;

import static seedu.address.logic.commands.CommandTestUtil.assertConfigCommandSuccess;
import static seedu.address.logic.commands.CommandTestUtil.assertConfigDiffCommandSuccess;
import static seedu.address.testutil.TypicalEvents.getTypicalEventBook;
import static seedu.address.testutil.TypicalPersons.getTypicalAddressBook;

import org.junit.Before;
import org.junit.Test;

import seedu.address.commons.core.Config;
import seedu.address.commons.core.Messages;
import seedu.address.logic.CommandHistory;
import seedu.address.logic.UndoRedoStack;
import seedu.address.model.Account;
import seedu.address.model.Model;
import seedu.address.model.ModelManager;
import seedu.address.model.UserPrefs;

/**
 * Contains integration tests (interaction with the Model) and unit tests for ListCommand.
 */
public class SetThemeCommandTest {

    private Config config;
    private Config expectedConfig;
    private SetThemeCommand setThemeCommand;
    private SetThemeCommand setThemeCommand2;
    private SetThemeCommand setThemeCommand3;
    private Model model = new ModelManager(getTypicalAddressBook(), getTypicalEventBook(), new UserPrefs(), new
            Account());

    @Before
    public void setUp() {
        config = new Config();
        expectedConfig = new Config();

        setThemeCommand = new SetThemeCommand();
        setThemeCommand.setData(model, new CommandHistory(), new UndoRedoStack(), config);
        setThemeCommand2 = new SetThemeCommand("nonsense");
        setThemeCommand2.setData(model, new CommandHistory(), new UndoRedoStack(), config);
        setThemeCommand3 = new SetThemeCommand("winter");
        setThemeCommand3.setData(model, new CommandHistory(), new UndoRedoStack(), config);
    }

    @Test
    public void execute_defaultTheme() {
        assertConfigCommandSuccess(setThemeCommand, config,
                String.format(SetThemeCommand.MESSAGE_CHANGED_THEME_SUCCESS, "summer"), expectedConfig);
    }

    @Test
    public void execute_nonsenseTheme() {
        assertConfigCommandSuccess(setThemeCommand2, config,
                String.format(Messages.MESSAGE_WRONG_THEME, "nonsense"), expectedConfig);
    }

    @Test
    public void execute_winterTheme() throws Exception {
        assertConfigDiffCommandSuccess(setThemeCommand3, config,
                String.format(SetThemeCommand.MESSAGE_CHANGED_THEME_SUCCESS, "winter"), expectedConfig);
    }
}
```
###### \java\seedu\address\ui\ViewAliasWindowTest.java
``` java
package seedu.address.ui;

import org.junit.Before;
import org.testfx.api.FxToolkit;

import guitests.guihandles.ViewAliasWindowHandle;
import javafx.stage.Stage;
import seedu.address.logic.Logic;

public class ViewAliasWindowTest extends GuiUnitTest {

    private ViewAliasWindow viewAliasWindow;
    private ViewAliasWindowHandle viewAliasWindowHandle;

    @Before
    public void setUp(Logic logic) throws Exception {

        guiRobot.interact(() -> viewAliasWindow = new ViewAliasWindow(logic.getCommands(), logic));
        Stage helpWindowStage = FxToolkit.setupStage((stage) -> stage.setScene(viewAliasWindow.getRoot().getScene()));
        FxToolkit.showStage();
        viewAliasWindowHandle = new ViewAliasWindowHandle(helpWindowStage);
    }
}
```