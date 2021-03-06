<script lang="javascript">
  import { fade } from "svelte/transition";
  import { state, projects, chapters, scenes, settings } from "../stores";
  import { deskgap } from "../utils";
  import { _ } from "svelte-i18n";

  import CreateProject from "./Overview/CreateProject.svelte";
  import ProjectOverview from "./Overview/Project.svelte";
  import Modal from "../shared/Modal.svelte";
  import moment from "moment";
  import "moment/locale/de";

  moment.locale($settings.language);

  let showCreateProject = false;
  let showEditProject = false;

  function changeProject(project) {
    state.setCurrentProject(project);
    projects.updateProjectTimestamp(project);
    deskgap.reload();
  }

  function setProjectTitle(project) {
    projects.setProjectTitle(
      project,
      document.getElementById("newProjectTitle").value
    );
    showEditProject = false;
  }

  function removeProject(project) {
    let confirmed = confirm($_("overview.project.confirmDelete"));
    if (confirmed == true) {
      $chapters
        .filter(ch => ch.project == project)
        .forEach(chapter => {
          scenes.removeAllScenes(chapter.id);
        });
      chapters.removeAllChapters(project);
      projects.removeProject(project);

      scenes.removeAllScenes(project);
      state.setCurrentProject("");
      deskgap.reload();
    }
  }

  function sort(b, a) {
    if (a.lastOpen < b.lastOpen) {
      return -1;
    }
    if (a.lastOpen > b.lastOpen) {
      return 1;
    }
    return 0;
  }
</script>

<style>
  .projectTitle {
    cursor: pointer;
  }

  .projectTitle:hover {
    text-decoration: underline;
    text-decoration-style: dashed;
  }

  .overview {
    max-width: 800px;
    margin: auto;
    text-align: center;
  }
</style>

<CreateProject
  bind:showCreateProject
  on:changeProject={event => changeProject(event.detail.project)} />

<div in:fade={{ duration: 100 }} class="overview">
  {#each $projects.filter(project => project.id == $state.currentProject) as project}
    <h1>{project.title}</h1>
    <ProjectOverview />
    <h3 class="projectTitle" on:click={() => (showEditProject = true)}>
      <span class="lnr lnr-cog" />
      Edit project
    </h3>
    <Modal bind:show={showEditProject}>
      <div class="field">
        <label class="big" for="author">{$_('overview.project.title')}:</label>
        <input
          id="newProjectTitle"
          type="text"
          autocomplete="off"
          bind:value={project.title} />
      </div>
      <div class="btn-group">
        <button on:click={() => setProjectTitle(project.id)}>
          {$_('overview.project.save')}
        </button>
        <button
          on:click={() => removeProject(project.id)}
          style="float: right;"
          class="warning">
          {$_('overview.project.delete')}
        </button>
      </div>
    </Modal>
  {/each}
  <h1>{$_('overview.projects.title')}</h1>
  <div class="grid">
    <div class="new" on:click={() => (showCreateProject = true)}>
      <span class="lnr lnr-plus-circle" />
    </div>
    {#each $projects.sort(sort) as project}
      <div on:click={() => changeProject(project.id)}>
        <h2>{project.title}</h2>
        <p>
          {$_('overview.projects.opened')}
          {moment(project.lastOpen, 'X').fromNow()}
        </p>
        <p>
          {$_('overview.project.chapters')}: {$chapters.filter(n => n.project == project.id).length}
        </p>
      </div>
    {/each}
  </div>
</div>
