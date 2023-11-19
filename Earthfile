VERSION 0.6
FROM fedora:39
ARG DEVCONTAINER_IMAGE_NAME_DEFAULT=ghcr.io/andyli/fedora_packaging_devcontainer

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG TARGETARCH

devcontainer-library-scripts:
    RUN curl -fsSLO https://raw.githubusercontent.com/andyli/vscode-dev-containers/fedora/script-library/common-redhat.sh
    RUN curl -fsSLO https://raw.githubusercontent.com/andyli/vscode-dev-containers/fedora/script-library/docker-redhat.sh
    SAVE ARTIFACT --keep-ts *.sh AS LOCAL .devcontainer/library-scripts/

devcontainer-base:
    # Avoid warnings by switching to noninteractive
    ENV DEBIAN_FRONTEND=noninteractive

    ARG INSTALL_ZSH="false"
    ARG UPGRADE_PACKAGES="true"
    ARG INSTALL_OH_MYS="false"
    ARG ADD_NON_FREE_PACKAGES="false"
    ARG ENABLE_NONROOT_DOCKER="true"
    COPY .devcontainer/library-scripts/common-redhat.sh .devcontainer/library-scripts/docker-redhat.sh /tmp/library-scripts/
    RUN /bin/bash /tmp/library-scripts/common-redhat.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "${INSTALL_OH_MYS}" "${ADD_NON_FREE_PACKAGES}"
    RUN /bin/bash /tmp/library-scripts/docker-redhat.sh "${ENABLE_NONROOT_DOCKER}" "/var/run/docker-host.sock" "/var/run/docker.sock" "${USERNAME}"

    RUN dnf install -y mock fedora-packager fedora-review
    RUN usermod -a -G mock "${USERNAME}"
    
    RUN dnf install -yq direnv bash-completion

    # Setting the ENTRYPOINT to docker-init.sh will configure non-root access 
    # to the Docker socket. The script will also execute CMD as needed.
    ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
    CMD [ "sleep", "infinity" ]

    RUN mkdir -m 777 "/workspace"
    WORKDIR /workspace

# Usage:
# COPY +earthly/earthly /usr/local/bin/
# RUN earthly bootstrap --no-buildkit --with-autocomplete
earthly:
    FROM +devcontainer-base
    ARG TARGETARCH
    ARG VERSION=0.7.13 # https://github.com/earthly/earthly/releases
    RUN curl -fsSL https://github.com/earthly/earthly/releases/download/v${VERSION}/earthly-linux-${TARGETARCH} -o /usr/local/bin/earthly \
        && chmod +x /usr/local/bin/earthly
    SAVE ARTIFACT /usr/local/bin/earthly

devcontainer:
    FROM +devcontainer-base

    # Install earthly
    COPY +earthly/earthly /usr/local/bin/
    RUN earthly bootstrap --no-buildkit --with-autocomplete

    USER $USERNAME

    # Config direnv
    COPY --chown=$USER_UID:$USER_GID .devcontainer/direnv.toml /home/$USERNAME/.config/direnv/config.toml
    RUN echo 'eval "$(direnv hook bash)"' >> ~/.bashrc

    USER root

    ARG GIT_SHA
    ENV GIT_SHA="$GIT_SHA"
    ARG IMAGE_NAME="$DEVCONTAINER_IMAGE_NAME_DEFAULT"
    ARG IMAGE_TAG="master"
    ARG IMAGE_CACHE="$IMAGE_NAME:$IMAGE_TAG"
    SAVE IMAGE --cache-from="$IMAGE_CACHE" --push "$IMAGE_NAME:$IMAGE_TAG"

ci-images:
    ARG --required GIT_REF_NAME
    ARG --required GIT_SHA
    BUILD +devcontainer \ 
        --IMAGE_CACHE="$DEVCONTAINER_IMAGE_NAME_DEFAULT:$GIT_REF_NAME" \
        --IMAGE_TAG="$GIT_REF_NAME" \
        --IMAGE_TAG="$GIT_SHA" \
        --GIT_SHA="$GIT_SHA"
