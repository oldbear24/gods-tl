<script>
  import PocketBase from "pocketbase";
  const pb = new PocketBase(window.location.origin);
  let attendaceData = new Array();
  let currentPeriod;
  let newPlayerName = "";
  let searchString=""
  function LogIn() {
    pb.collection("users").authWithOAuth2({ provider: "discord" });
  }
  async function getAttendace() {
    await getLatestPeriod();
    console.log(currentPeriod);

let filter = "period_id = '" + currentPeriod.id + "' && kicked=false"
if(searchString!=""){
  filter+=" && name ~ '"+searchString+"%'"
}

    attendaceData = await pb
      .collection("playerAttendanceView")
      .getFullList({
        filter: filter,
        sort: "id"
      });
    console.log(attendaceData);
  }
  async function getLatestPeriod() {
    let pers = await pb.collection("currentPeriod").getFullList();
    currentPeriod = pers[0];
    if (currentPeriod==undefined){
      const data ={
        "start": new Date(),
        "periodNumber":1
      }
      currentPeriod = pb.collection("periods").create(data)
    }
  }

  async function setNewAttId(count, id, playerId) {
    if (id == "") {
      const data = {
        player: playerId,
        period: currentPeriod.id,
        attendanceCount: count,
      };
      await pb.collection("periodPlayer").create(data);
    } else {
      const data = {
        attendanceCount: count,
      };
      await pb.collection("periodPlayer").update(id, data);
    }

    getAttendace();
  }

  async function createNewPeriod() {
    if (!window.confirm("Do you really want to create new period?")) {
      return
    }
    let date = new Date();
    const data2 = {
      start: date,
      periodNumber: currentPeriod.periodNumber + 1,
    };
    await pb.collection("periods").create(data2);
    const data = {
      end: date,
    };
    await pb.collection("periods").update(currentPeriod.id, data);

    getAttendace();
  }

  async function handleSubmit(event) {
    event.preventDefault();

    const data = {
      name: newPlayerName,
    };

    try {
      await pb.collection("players").create(data);
      // Clear form inputs after submission
      newPlayerName = "";
      getAttendace();
    } catch (error) {
      console.log(error);
    }
  }
  async function kickPlayer(id) {
    const data = {
      leaved: true,
    };

    await pb.collection("players").update(id, data);
    getAttendace();
  }
  getAttendace();
</script>
<div class="navbar bg-base-100">
  <div class="flex-1">
    <a class="btn btn-ghost text-xl">GoDS TL Attendace</a>
    <button on:click={() => createNewPeriod()} class="btn btn-ghost"
      >New period</button
    >
  </div>
  <div class="flex">
    <button on:click={() => LogIn()} class="btn btn-square btn-ghost">
      Log In
    </button>
  </div>
</div>
<input bind:value={searchString} type="text" placeholder="Search by name" class="input input-bordered w-full max-w-xs" />
<button on:click={()=>getAttendace()} class="btn">Search</button>
<div class="overflow-x-auto">
  <table class="table">
    <!-- head -->
    <thead>
      <tr>
        <th>Name</th>
        <th>Attendance</th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      {#each attendaceData as att}
        <tr>
          <td>{att.name}</td>
          <td>
            <div class="join">
              <button
                class="join-item btn"
                on:click={() =>
                  setNewAttId(
                    att.attendanceCount + 1,
                    att.per_pla_id,
                    att.player_id,
                  )}>+</button
              >
              <button class="join-item btn">{att.attendanceCount}</button>
              <button
                class="join-item btn"
                on:click={() =>
                  setNewAttId(
                    att.attendanceCount - 1,
                    att.per_pla_id,
                    att.player_id,
                  )}>-</button
              >
            </div>
          </td>
          <td
            ><button
              on:click={() => kickPlayer(att.player_id)}
              class="btn btn-s btn-error">Kick</button
            ></td
          >
        </tr>
      {/each}
    </tbody>
  </table>
</div>
<div class="p-6 max-w-sm mx-auto rounded-xl shadow-md space-y-4">
  <form on:submit={handleSubmit} class="form-control space-y-4">
    <label class="label">
      <span class="label-text">Player Name</span>
      <input
        type="text"
        bind:value={newPlayerName}
        placeholder="Enter player name"
        class="input input-bordered w-full"
        required
      />
    </label>
    <button type="submit" class="btn btn-primary w-full">Create Player</button>
  </form>
</div>
