# proto-preview

## task
    use fixed name for preview image
    user always must see actual preview version
        web server set caching http-headers: browser check if new version is available before using cache
            https://requestmetrics.com/web-performance/http-caching/

## why
    I think fixed name for preview is more simple implementation 
    than 
        generating new name for preview on every content update
        storing this name in table email.preview-path, 
        sending to client
        removing previous preview file

## steps
1) implementation with simple web-server 
    (expressjs or fastify)

2) implementation with cloud platform  
    (aws, s3 for content file, preview file, lambda function to generate preview)

## components  
    server api:  
        /email, 
        /email{id} create/update
            on create/update generate preview
        file storage, in file system (external volume for container)
            {id}/content.html, preview.png
            or
            /content/content-{id}.html -> {id}/content.html This catalog can NOT be deleted
            /preview/preview-{id}.png -> {id}/preview.png. This catalog can be deleted and regenerated
    db: 
        table email, fields: id, name?, onContentUpdate, onPreviewUpdate
    web-ui
        /email -> list of emails with actual preview
        /email/{id} -> edit content
            left part of page: edit content
                save content periodicaly,
                save content on close page
                    isActual = content.length == prevContent.length && hash(content) == prevContentHash
                    isChanged = !isActual
                    isChanged = content.length !== prevContent.length || hash(content) !== prevContentHash
            right part of page: actual preview

            client must get message(WebSockets?) from server new preview is ready: do update

## tests  
    puppeteer
        user see actual preview after changes
        ?generate qr-code image, read qr-code in test
    superagent
        check http caching header for /email/{id}/preview.png

## logic  
    on update content
        table email.onContentUpdate = now
        start async generatePreview(emailId)
            const onContentUpdate = email(emailId).onContentUpdate
            // generate preview and store
            // transaction
            if email(emailId).onPreviewUpdate < onContentUpdate
                save email(emailId).onPreviewUpdate = onContentUpdate

    isPreviewActual = onPreviewUpdate === onContentUpdate

## difficulties  
    client get actual version after update, not from cache

    update preview on editing in web-ui /email/{id}

    two updates: update1, update2, 
        update1 is before update2
        update1 takes more time for preview generation than update2 and finished after update2
            update1 must not rewrite changes of update2

    I can block item(email(id).isBlocked) on time of preview generation or time of replacing file
    

## extra functionality  
    store content versions in git repo   
        restore from previous version  
        compare content versions  
        store version list in table email-editions: emailId, origin (user, autosave?, restored from versionId, commitHash): , commitHash  

    this functionality require uniq names for preview for each version  
        ?do not store this preview permanently?  
            /{id}/version/{versionId}/preview  
            or /{id}/version-preview-{versionId}.png  
            or /{id}/versions/preview-{versionId}.png  